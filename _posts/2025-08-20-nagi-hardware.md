---
title: Nagi hardware and control
author: andy
date: 2025-08-20 15:20:00 +/-0080
categories: [doc]
tags: [nagi]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---


## Hardware
### BoM and 3D printing
follow [xlerobot](https://xlerobot.readthedocs.io/en/latest/hardware/getting_started/3d.html)

### Assembly
follow [lekiwi](https://github.com/SIGRobotics-UIUC/LeKiwi/blob/main/Assembly.md) assembly processes 

## Control
### Requirements
Install lerobot 

```bash
pip install 'lerobot[feetech]'
```

To ssh into the robot, first check its ip address by the following command in a terminal.
```bash
# suppose the robot ip is under this subnet: 192.168.1.x
sudo nmap -sn 192.168.1.0/24
```

### Setup motor ID
connect to each motor individually, and change their motor IDs using the following code.

```python
"""
Change the ID of a single Feetech STS/SMS wheel servo.

Examples:
  python3 write_servo_id.py --port /dev/ttyACM0 --baud 1000000 --new-id 9
  python3 write_servo_id.py --port COM5 --baud 1000000 --new-id 12 --old-id 1
  python3 write_servo_id.py --new-id 7 --scan 0-253
"""

import time
import argparse
from STservo_sdk import PortHandler, sts, COMM_SUCCESS

# Control table (Feetech STS/SMS)
ADDR_ID   = 5
ADDR_LOCK = 55

INSTR_WRITE = 0x03

def checksum(core_bytes):  # sum from ID..last param, ones' complement
    return (~sum(core_bytes) & 0xFF) & 0xFF

def make_write_packet(target_id, addr, data_bytes):
    # Packet: FF FF | ID | LEN | INST | ADDR | data... | CHK
    params = [addr] + list(data_bytes)
    length = 2 + len(params)            # INST + params
    core   = [target_id, length, INSTR_WRITE] + params
    pkt    = bytes([0xFF, 0xFF] + core + [checksum(core)])
    return pkt

def write_byte(port, target_id, addr, value):
    pkt = make_write_packet(target_id, addr, [value & 0xFF])
    n = port.writePort(pkt)             # STservo_sdk PortHandler.writePort(bytes) -> int
    time.sleep(0.02)
    return n == len(pkt)

def ping(proto, sid, retries=2, delay=0.02):
    for _ in range(max(1, retries)):
        model, comm, err = proto.ping(sid)
        if comm == COMM_SUCCESS:
            return True
        time.sleep(delay)
    return False

def parse_scan(spec: str):
    # "0-31" -> range(0, 32); "1" -> range(1, 2)
    if "-" in spec:
        a, b = spec.split("-", 1)
        a, b = int(a), int(b)
        if a > b: a, b = b, a
        return range(a, b + 1)
    v = int(spec)
    return range(v, v + 1)

def find_one(proto, id_range):
    responders = [sid for sid in id_range if ping(proto, sid)]
    return responders if responders else []

def main():
    ap = argparse.ArgumentParser(description="Change Feetech STS/SMS servo ID safely.")
    ap.add_argument("--port", default="/dev/ttyACM0", help="Serial port (e.g., /dev/ttyACM0 or COM5)")
    ap.add_argument("--baud", type=int, default=1_000_000, help="Baud rate (default 1,000,000)")
    ap.add_argument("--new-id", type=int, required=True, help="New ID 0..253")
    ap.add_argument("--old-id", type=int, help="Known current ID; skip bus scan")
    ap.add_argument("--scan", default="0-31", help="ID scan range when --old-id not given (e.g., 0-253)")
    args = ap.parse_args()

    if not (0 <= args.new_id <= 253):
        raise SystemExit("new-id must be 0..253")
    if args.old_id is not None and not (0 <= args.old_id <= 253):
        raise SystemExit("old-id must be 0..253")

    port = PortHandler(args.port)
    if not port.openPort():
        raise SystemExit(f"Failed to open {args.port}")
    try:
        if not port.setBaudRate(args.baud):
            raise SystemExit(f"Failed to set baud {args.baud}")

        proto = sts(port)

        # 1) Determine exactly one target ID
        if args.old_id is None:
            ids = find_one(proto, parse_scan(args.scan))
            if len(ids) != 1:
                raise SystemExit(
                    f"Refusing to proceed: found {len(ids)} responders {ids}. "
                    f"Ensure only one wheel is connected, or specify --old-id."
                )
            old_id = ids[0]
        else:
            if not ping(proto, args.old_id):
                raise SystemExit(f"No response from --old-id {args.old_id} at baud {args.baud}.")
            old_id = args.old_id

        print(f"Found servo at ID {old_id}")

        # 2) Unlock EEPROM (LOCK=0)
        unlocked = False
        if hasattr(proto, "unLockEprom"):
            try:
                proto.unLockEprom(old_id); unlocked = True
            except Exception:
                unlocked = False
        if not unlocked:
            if not write_byte(port, old_id, ADDR_LOCK, 0):
                raise SystemExit("Failed to clear LOCK (unlock EEPROM).")

        # 3) Write new ID
        if not write_byte(port, old_id, ADDR_ID, args.new_id):
            raise SystemExit("Write to ID register failed (no bytes written).")
        time.sleep(0.06)

        # 4) Lock EEPROM again with NEW ID (LOCK=1)
        relocked = False
        if hasattr(proto, "LockEprom"):
            try:
                proto.LockEprom(args.new_id); relocked = True
            except Exception:
                relocked = False
        if not relocked:
            if not write_byte(port, args.new_id, ADDR_LOCK, 1):
                print("Warning: could not re-lock EEPROM via raw write.")

        time.sleep(0.05)

        # 5) Verify
        ok = ping(proto, args.new_id)
        # Optional: verify by reading back ADDR_ID if SDK supports it
        id_match = "unknown"
        if hasattr(proto, "readByte"):
            try:
                rb, comm, err = proto.readByte(args.new_id, ADDR_ID)
                if comm == COMM_SUCCESS:
                    id_match = rb
            except Exception:
                pass

        if ok:
            extra = f" (ID register now {id_match})" if isinstance(id_match, int) else ""
            print(f"✅ Success: ID {old_id} → {args.new_id}{extra}")
        else:
            print(
                "⚠️ New ID didn't answer to ping. "
                f"If Reply/Status packets are disabled, try moving ID {args.new_id} or power-cycling. "
                "Also confirm baud rate and wiring."
            )

    finally:
        try:
            port.closePort()
        except Exception:
            pass

if __name__ == "__main__":
    main()
```

### Motion Control
Use the following code to do forward kinematics for the car.

```python
#!/usr/bin/env python3
from __future__ import annotations
from time import sleep, monotonic
import math
from typing import Dict, Iterable
from lerobot.motors import Motor, MotorNormMode
from lerobot.motors.feetech import FeetechMotorsBus

class LeKiwiDrive:
    """
    Minimal LeRobot-based driver for a 3-omni LeKiwi base (Feetech STS/STS3215 wheels).
    Provides:
      - connect()/disconnect()
      - drive(vx, vy, omega)  # m/s, m/s, rad/s
      - forward()/backward()/turn()
      - stop()
      - teleop_keyboard()     # WASD/arrow keys

    Conventions:
      Body frame: +x = right, +y = forward, +omega = CCW (rad/s).
      Wheel layout (top view):
            LF(7)
           /   \
          *     *  RF(9)
            \ /
             *     Rear(8)

    Kinematics (3-omni standard):
      u_i = ( -sin(phi_i) * vx + cos(phi_i) * vy + d * omega ) / r
      where r = wheel radius (m), d = chassis center-to-wheel distance (m),
      and phi_i are wheel drive-axis angles: LF=+60°, Rear=180°, RF=-60°.
    """

    def __init__(
        self,
        port: str = "/dev/ttyACM0",
        ids: Dict[str, int] = {"lf": 7, "rear": 8, "rf": 9},
        model: str = "sts3215",
        r_m: float = 0.040,     # wheel radius (m) — adjust to your wheels
        d_m: float = 0.115,     # center-to-wheel distance (m) — adjust to your chassis
        speed_scale: float = 1.0,   # raw bus units per wheel rad/s (calibrate with calibrate_speed_scale)
        signs: Dict[str, int] = {"lf": 1, "rear": 1, "rf": 1},  # flip if any wheel spins the opposite direction
        limit_raw: int = 9000,  # software clamp for Goal_Velocity magnitude
    ):
        self.port = port
        self.ids = ids
        self.model = model
        self.r = r_m
        self.d = d_m
        self.k = float(speed_scale)
        self.signs = signs
        self.limit_raw = int(abs(limit_raw))

        # LeRobot bus & motor map (raw register units)
        self.bus = FeetechMotorsBus(
            port=self.port,
            motors={
                "lf":   Motor(self.ids["lf"],   self.model, MotorNormMode.RANGE_M100_100),
                "rear": Motor(self.ids["rear"], self.model, MotorNormMode.RANGE_M100_100),
                "rf":   Motor(self.ids["rf"],   self.model, MotorNormMode.RANGE_M100_100),
            },
            calibration=None,
        )

        # Wheel drive-axis angles (radians)
        self.phi = {"lf": math.radians(60.0), "rear": math.radians(180.0), "rf": math.radians(-60.0)}

    # ---------- lifecycle ----------
    def connect(self):
        self.bus.connect()
        # Put wheels in speed (wheel) mode and torque on.
        # Feetech docs: Operating_Mode=1 -> speed closed-loop (wheel). :contentReference[oaicite:1]{index=1}
        for name in ["lf", "rear", "rf"]:
            # Allow EEPROM writes if your firmware protects mode changes
            self.bus.write("Lock", name, 0)
            self.bus.write("Operating_Mode", name, 1)  # wheel speed closed-loop
            self.bus.write("Lock", name, 1)
            self.bus.write("Torque_Enable", name, 1)

    def disconnect(self, torque_off: bool = True):
        if torque_off:
            for name in ["lf", "rear", "rf"]:
                self.bus.write("Torque_Enable", name, 0)
        self.bus.disconnect()

    # ---------- calibration helper ----------
    def calibrate_speed_scale(self, test_raw: int = 800, sample_s: float = 0.8) -> float:
        """
        Estimate conversion k: raw_units_per_(rad/s).
        Sends 'test_raw' to each wheel (one-by-one), reads Present_Velocity (raw units),
        computes k = raw / measured_raw  (identity if table already returns raw).
        Adjust this method if your control table returns deg/s or steps/s.
        """
        ks = []
        for name in ["lf", "rear", "rf"]:
            self.bus.write("Goal_Velocity", name, int(test_raw))
            sleep(sample_s)
            pv = float(self.bus.read("Present_Velocity", name))
            if abs(pv) > 1e-6:
                ks.append(abs(test_raw) / abs(pv))
            self.bus.write("Goal_Velocity", name, 0)
            sleep(0.2)
        if ks:
            self.k = sum(ks) / len(ks)
        return self.k

    # ---------- motion API ----------
    def drive(self, vx: float, vy: float, omega: float):
        """
        Command a chassis twist (m/s, m/s, rad/s) continuously until stop() or another drive().
        """
        u = {}
        for name, phi in self.phi.items():
            # wheel angular velocity (rad/s)
            ui = (-math.sin(phi) * vx + math.cos(phi) * vy + self.d * omega) / self.r
            u[name] = self._to_raw(name, ui)

        self.bus.sync_write("Goal_Velocity", {"lf": u["lf"], "rear": u["rear"], "rf": u["rf"]})

    def stop(self):
        self.bus.sync_write("Goal_Velocity", {"lf": 0, "rear": 0, "rf": 0})

    def forward(self, distance_m: float, speed_m_s: float = 0.20):
        """Drive +y by distance at speed (simple time-based profile)."""
        assert speed_m_s > 0
        t = abs(distance_m) / speed_m_s if distance_m != 0 else 0
        self.drive(0.0, math.copysign(speed_m_s, distance_m), 0.0)
        self._sleep_s(t)
        self.stop()

    def backward(self, distance_m: float, speed_m_s: float = 0.20):
        """Drive -y by distance at speed."""
        self.forward(-abs(distance_m), speed_m_s)

    def turn(self, degrees: float, deg_per_s: float = 90.0):
        """Turn in place by +CCW degrees at a given rotational speed."""
        assert deg_per_s > 0
        omega = math.radians(math.copysign(deg_per_s, degrees))
        t = abs(degrees) / deg_per_s
        self.drive(0.0, 0.0, omega)
        self._sleep_s(t)
        self.stop()

    # ---------- keyboard teleop ----------
    def teleop_keyboard(self, base_linear=0.25, base_turn_dps=120.0, hz=30):
        """
        Keyboard teleop: W/S or ↑/↓ for forward/back, A/D or ←/→ for left/right turn,
        Space=stop, Q=quit. Shift doubles speed. Z/X adjust linear; C/V adjust turn rate.
        Uses curses for non-blocking input and safe teardown. :contentReference[oaicite:2]{index=2}
        """
        import curses, math

        def _ui(stdscr):
            curses.curs_set(0)
            stdscr.keypad(True)
            stdscr.timeout(int(1000 / hz))   # non-blocking with short timeout
            vy = 0.0
            omega = 0.0
            lin = float(base_linear)
            yaw = math.radians(float(base_turn_dps))

            # make sure in wheel mode & torque-enabled
            for name in ["lf", "rear", "rf"]:
                self.bus.write("Operating_Mode", name, 1)
                self.bus.write("Torque_Enable", name, 1)

            running = True
            while running:
                ch = stdscr.getch()
                if ch != -1:
                    if ch in (ord('w'), ord('W'), curses.KEY_UP):
                        vy = 2*lin if ch == ord('W') else lin; omega = 0.0
                    elif ch in (ord('s'), ord('S'), curses.KEY_DOWN):
                        vy = -(2*lin if ch == ord('S') else lin); omega = 0.0
                    elif ch in (ord('a'), ord('A'), curses.KEY_LEFT):
                        omega =  2*yaw if ch == ord('A') else yaw; vy = 0.0
                    elif ch in (ord('d'), ord('D'), curses.KEY_RIGHT):
                        omega = -(2*yaw if ch == ord('D') else yaw); vy = 0.0
                    elif ch in (ord('z'), ord('Z')):
                        lin = max(0.05, lin * 0.8)
                    elif ch in (ord('x'), ord('X')):
                        lin = min(1.5, lin / 0.8)
                    elif ch in (ord('c'), ord('C')):
                        yaw = max(math.radians(10), yaw * 0.8)
                    elif ch in (ord('v'), ord('V')):
                        yaw = min(math.radians(540), yaw / 0.8)
                    elif ch == ord(' '):
                        vy = 0.0; omega = 0.0
                    elif ch in (ord('q'), ord('Q')):
                        running = False

                # send current command
                self.drive(0.0, vy, omega)

                stdscr.erase()
                stdscr.addstr(0, 0, "LeKiwi Keyboard Teleop  |  W/S or ↑/↓: forward/back   A/D or ←/→: turn   Space: stop   Q: quit")
                stdscr.addstr(2, 0, f"linear={lin:.3f} m/s   yaw={math.degrees(yaw):.0f} deg/s")
                stdscr.addstr(3, 0, f"cmd: vy={vy:+.3f} m/s   omega={math.degrees(omega):+6.1f} deg/s")
                stdscr.refresh()

            # on exit, stop & torque off
            self.stop()
            for name in ["lf", "rear", "rf"]:
                self.bus.write("Torque_Enable", name, 0)

        curses.wrapper(_ui)

    # ---------- utilities ----------
    def _to_raw(self, name: str, wheel_rad_s: float) -> int:
        """Convert wheel rad/s to Feetech raw Goal_Velocity, apply per-wheel sign & clamp."""
        raw = int(self.signs[name] * self.k * wheel_rad_s)
        return max(-self.limit_raw, min(self.limit_raw, raw))

    @staticmethod
    def _sleep_s(t: float):
        deadline = monotonic() + t
        while monotonic() < deadline:
            sleep(0.01)

# ------------------------- example -------------------------
if __name__ == "__main__":
    drv = LeKiwiDrive(
        port="/dev/ttyACM0",
        ids={"lf": 7, "rear": 8, "rf": 9},
        r_m=0.040, d_m=0.115,
        speed_scale=1.0,         # run calibrate_speed_scale() to refine SI mapping
        signs={"lf": 1, "rear": 1, "rf": 1},
        limit_raw=9999,
    )
    drv.connect()
    try:
        # Basic motions
        drv.forward(0.5, speed_m_s=0.25)
        drv.backward(0.5, speed_m_s=0.25)
        drv.turn(90, deg_per_s=180)

        # Or drive from keyboard:
        # drv.teleop_keyboard(base_linear=0.25, base_turn_dps=120, hz=30)
    finally:
        drv.stop()
        drv.disconnect()
```




## Reference
- [Xlerobot](https://xlerobot.readthedocs.io/en/latest/index.html)
- [lerobot](https://github.com/huggingface/lerobot)


