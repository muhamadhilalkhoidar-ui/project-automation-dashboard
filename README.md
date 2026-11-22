#!/usr/bin/env python3
"""
Generator footprint KiCad (.kicad_mod) untuk skematik:
ESP32 DevKitC 38-pin, OLED SSD1306 I2C, TP4056, MT3608, JST XH 2P, header 1x04, LiPo 2 pin.
Jalankan: python3 gen_footprints.py
"""
import os

OUT = "ESP32_Project.pretty"
os.makedirs(OUT, exist_ok=True)
PITCH = 2.54


def pad(num, x, y, size, drill, shape="circle"):
    return (f'  (pad "{num}" thru_hole {shape} (at {x} {y}) '
            f'(size {size[0]} {size[1]}) (drill {drill}) (layers "*.Cu" "*.Mask"))')


def rect(x1, y1, x2, y2, layer, width=0.12):
    return (f'  (fp_rect (start {x1} {y1}) (end {x2} {y2}) '
            f'(layer "{layer}") (width {width}) (fill none))')


def write_fp(name, descr, pads, silk, crt, ref_y, val_y):
    lines = [
        f'(footprint "{name}" (version 20211014) (generator "gen_footprints")',
        '  (layer "F.Cu")',
        f'  (descr "{descr}")',
        '  (attr through_hole)',
        f'  (fp_text reference "REF**" (at 0 {ref_y}) (layer "F.SilkS") '
        f'(effects (font (size 1 1) (thickness 0.15))))',
        f'  (fp_text value "{name}" (at 0 {val_y}) (layer "F.Fab") '
        f'(effects (font (size 1 1) (thickness 0.15))))',
        rect(*silk, "F.SilkS"),
        rect(*crt, "F.CrtYd", 0.05),
    ] + pads + [')']
    with open(os.path.join(OUT, name + ".kicad_mod"), "w") as f:
        f.write("\n".join(lines) + "\n")
    print("OK:", name)


def pin_header(n):
    """Header 1xN vertikal, pitch 2.54 mm, pin 1 kotak."""
    pads = [pad(1, 0, 0, (1.7, 1.7), 1.0, "rect")]
    pads += [pad(i + 1, 0, round(i * PITCH, 3), (1.7, 1.7), 1.0) for i in range(1, n)]
    bottom = round((n - 1) * PITCH, 3)
    write_fp(
        name=f"PinHeader_1x{n:02d}_P2.54mm_Vertical",
        descr=f"Pin header 1x{n:02d}, 2.54 mm pitch, vertical",
        pads=pads,
        silk=(-1.27, -1.27, 1.27, bottom + 1.27),
        crt=(-1.75, -1.75, 1.75, bottom + 1.75),
        ref_y=-2.5,
        val_y=bottom + 2.5,
    )


def jst_xh_2p():
    """JST XH 2 pin vertikal, pitch 2.50 mm."""
    pads = [
        pad(1, 0, 0, (1.7, 1.95), 0.9, "rect"),
        pad(2, 2.5, 0, (1.7, 1.95), 0.9, "oval"),
    ]
    write_fp(
        name="JST_XH_B2B-XH-A_1x02_P2.50mm_Vertical",
        descr="JST XH 2 pin vertical, 2.50 mm pitch",
        pads=pads,
        silk=(-1.6, -2.4, 4.1, 2.2),
        crt=(-2.0, -2.8, 4.5, 2.6),
        ref_y=-3.2,
        val_y=3.0,
    )


def esp32_devkitc_38():
    """ESP32 DevKitC 38 pin: 2 baris 19 pin, pitch 2.54, jarak baris 25.4 mm."""
    row_gap = 25.4
    pads = []
    for i in range(19):
        y = round(i * PITCH, 3)
        shape_l = "rect" if i == 0 else "circle"
        pads.append(pad(i + 1, 0, y, (1.7, 1.7), 1.0, shape_l))          # kiri: 1..19
        pads.append(pad(20 + i, row_gap, y, (1.7, 1.7), 1.0))             # kanan: 20..38
    bottom = round(18 * PITCH, 3)
    write_fp(
        name="ESP32_DevKitC_38_2x19_P2.54mm",
        descr="ESP32 DevKitC 38-pin, 2x19, 2.54 mm pitch, row spacing 25.4 mm",
        pads=pads,
        silk=(-2.6, -2.6, row_gap + 2.6, bottom + 2.6),
        crt=(-3.0, -3.0, row_gap + 3.0, bottom + 3.0),
        ref_y=-3.5,
        val_y=bottom + 3.5,
    )


if __name__ == "__main__":
    jst_xh_2p()                  # J1..J5 (JST_XH_2P)
    pin_header(4)                # J6 (Conn_01x04) dan DS1 OLED (4 pin)
    pin_header(2)                # BT1 LiPo 2000mAh (2 pin)
    pin_header(4)                # U3 MT3608 (4 pin, sesuaikan dengan modul)
    pin_header(6)                # U2 TP4056 (6 pin, sesuaikan dengan modul)
    esp32_devkitc_38()           # U1 ESP32 DevKitC
