# ley_enfriamiento_newton
como determinar la hora de muerte de una persona 
"""
=============================================================
  LEY DE ENFRIAMIENTO DE NEWTON — Determinación Hora de Muerte
  Modelo: T(t) = T_A + C * e^(k*t)
************************************************************************
Datos del problema:
  - T_A = 22°C  (temperatura del ambiente)
  - t=0  (12:30 PM): T = 30°C  → cadáver encontrado
  - t=1h (13:30 PM): T = 28°C  → segunda medición
  - T_muerte = 37°C             → temperatura normal del cuerpo humano
  - Objetivo: hallar t < 0 (hora de muerte)
************************************************************************
"""

import numpy as np
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
from matplotlib.gridspec import GridSpec
from datetime import datetime, timedelta

T_A   = 22.0   # Temperatura ambiente (°C)
T0    = 30.0   # Temperatura del cadáver a las 12:30 PM
T1    = 28.0   # Temperatura del cadáver a la 1:30 PM (t=1h)
T_mrt = 37.0   # Temperatura normal del cuerpo humano (muerte)
t1    = 1.0    # Delta de tiempo entre mediciones (horas)

HORA_HALLAZGO = datetime(2024, 1, 1, 12, 30)  # 12:30 PM (referencia)

# ── PASO 1: CALCULAR C ───────────────────────────────────────
# T(0) = T_A + C  →  C = T(0) - T_A
C = T0 - T_A
print("=" * 55)
print("  LEY DE ENFRIAMIENTO DE NEWTON")
print("=" * 55)
print(f"\n[1] Cálculo de C:")
print(f"    T(0) = T_A + C")
print(f"    {T0} = {T_A} + C  →  C = {C:.4f} °C")

# ── PASO 2: CALCULAR k ───────────────────────────────────────
# T(1) = T_A + C*e^k  →  k = ln((T(1) - T_A) / C)
k = np.log((T1 - T_A) / C)
print(f"\n[2] Cálculo de k:")
print(f"    T(1) = T_A + C·e^k")
print(f"    {T1} = {T_A} + {C}·e^k")
print(f"    e^k = ({T1} - {T_A}) / {C} = {(T1-T_A)/C:.4f}")
print(f"    k   = ln({(T1-T_A)/C:.4f}) = {k:.6f} h⁻¹")

# ── PASO 3: HORA DE MUERTE ────────────────────────────────────
# T_mrt = T_A + C·e^(k*t_mrt)  →  t_mrt = ln((T_mrt - T_A)/C) / k
t_mrt = np.log((T_mrt - T_A) / C) / k
horas  = int(abs(t_mrt))
minutos = int(round((abs(t_mrt) - horas) * 60))
hora_muerte = HORA_HALLAZGO + timedelta(hours=t_mrt)

print(f"\n[3] Hora de muerte:")
print(f"    T_muerte = T_A + C·e^(k·t)")
print(f"    {T_mrt} = {T_A} + {C}·e^({k:.4f}·t)")
print(f"    t = ln(({T_mrt} - {T_A})/{C}) / {k:.4f}")
print(f"    t = {t_mrt:.4f} horas  ({horas}h {minutos}min antes del hallazgo)")
print(f"\n  ✦ HORA DE MUERTE ESTIMADA: {hora_muerte.strftime('%I:%M %p')}")
print("=" * 55)

def T(t):
    return T_A + C * np.exp(k * t)

fig = plt.figure(figsize=(14, 9), facecolor="#0f0f1a")
gs  = GridSpec(2, 2, figure=fig, hspace=0.42, wspace=0.35)

ax1 = fig.add_subplot(gs[0, :])   # Curva principal (fila superior completa)
ax2 = fig.add_subplot(gs[1, 0])   # Sensibilidad a T_muerte
ax3 = fig.add_subplot(gs[1, 1])   # Residuos / verificación del modelo

COLOR_BG   = "#0f0f1a"
COLOR_GRID = "#2a2a3e"
COLOR_MAIN = "#4fc3f7"
COLOR_MORT = "#ef5350"
COLOR_AMB  = "#66bb6a"
COLOR_PTS  = "#ffd54f"
COLOR_TEXT = "#e0e0e0"

for ax in [ax1, ax2, ax3]:
    ax.set_facecolor("#15152a")
    ax.tick_params(colors=COLOR_TEXT, labelsize=9)
    ax.xaxis.label.set_color(COLOR_TEXT)
    ax.yaxis.label.set_color(COLOR_TEXT)
    ax.title.set_color(COLOR_TEXT)
    for spine in ax.spines.values():
        spine.set_edgecolor(COLOR_GRID)
    ax.grid(True, color=COLOR_GRID, linewidth=0.7, linestyle="--", alpha=0.6)

# ── AX1: CURVA PRINCIPAL ─────────────────────────────────────
t_range = np.linspace(t_mrt - 0.3, 3.5, 500)
ax1.plot(t_range, T(t_range), color=COLOR_MAIN, linewidth=2.5,
         label=r"$T(t) = 22 + 8\,e^{kt}$", zorder=5)

# Líneas de referencia
ax1.axhline(T_A,   color=COLOR_AMB,  linewidth=1.4, linestyle="--",
            label=f"Ambiente $T_A = {T_A}°C$", alpha=0.85)
ax1.axhline(T_mrt, color=COLOR_MORT, linewidth=1.4, linestyle="--",
            label=f"Temperatura de muerte = {T_mrt}°C", alpha=0.85)

# Puntos de medición
ax1.scatter([0, 1], [T0, T1], color=COLOR_PTS, s=90, zorder=10,
            label="Mediciones (12:30 y 13:30)")
ax1.annotate(f" 12:30 PM\n T={T0}°C", (0, T0), color=COLOR_PTS,
             fontsize=8.5, va="bottom")
ax1.annotate(f" 13:30 PM\n T={T1}°C", (1, T1), color=COLOR_PTS,
             fontsize=8.5, va="bottom")

# Punto de muerte
ax1.scatter([t_mrt], [T_mrt], color=COLOR_MORT, s=120, zorder=10,
            marker="*", label=f"Hora de muerte ≈ {hora_muerte.strftime('%I:%M %p')}")
ax1.axvline(t_mrt, color=COLOR_MORT, linewidth=1.1, linestyle=":", alpha=0.7)
ax1.annotate(f"  ← Muerte\n  {hora_muerte.strftime('%I:%M %p')}",
             (t_mrt, T_mrt - 0.5), color=COLOR_MORT, fontsize=9, va="top")

# Etiquetas de tiempo en x
xticks_h = np.arange(np.floor(t_mrt) - 0.5, 4, 0.5)
xlabels  = []
for th in xticks_h:
    hora_tick = HORA_HALLAZGO + timedelta(hours=float(th))
    xlabels.append(hora_tick.strftime("%H:%M"))
ax1.set_xticks(xticks_h)
ax1.set_xticklabels(xlabels, rotation=35, fontsize=8)
ax1.set_xlabel("Hora del día", fontsize=11)
ax1.set_ylabel("Temperatura (°C)", fontsize=11)
ax1.set_title("Ley de Enfriamiento de Newton — Determinación de Hora de Muerte",
              fontsize=13, fontweight="bold", pad=12)
ax1.legend(loc="upper right", fontsize=8.5, facecolor="#1e1e30",
           edgecolor=COLOR_GRID, labelcolor=COLOR_TEXT)

# ── AX2: SENSIBILIDAD T_muerte ───────────────────────────────
T_vals   = np.linspace(35, 40, 200)
t_vals   = np.log((T_vals - T_A) / C) / k
hora_vals = [HORA_HALLAZGO + timedelta(hours=float(tv)) for tv in t_vals]
t_decimal = [(HORA_HALLAZGO + timedelta(hours=float(tv))).hour +
             (HORA_HALLAZGO + timedelta(hours=float(tv))).minute / 60
             for tv in t_vals]

ax2.plot(T_vals, [tv for tv in t_vals], color="#ba68c8", linewidth=2)
ax2.axvline(T_mrt, color=COLOR_MORT, linestyle="--", linewidth=1.2, alpha=0.8)
ax2.axhline(t_mrt, color=COLOR_MORT, linestyle="--", linewidth=1.2, alpha=0.8)
ax2.scatter([T_mrt], [t_mrt], color=COLOR_MORT, s=80, zorder=10)
ax2.annotate(f" t = {t_mrt:.2f}h", (T_mrt, t_mrt),
             color=COLOR_TEXT, fontsize=8)
ax2.set_xlabel("T de muerte asumida (°C)", fontsize=9)
ax2.set_ylabel("t de muerte (h desde 12:30)", fontsize=9)
ax2.set_title("Sensibilidad: T_muerte vs. t estimado", fontsize=10)

# ── AX3: VERIFICACIÓN DEL MODELO ────────────────────────────
t_check  = np.array([t_mrt, 0.0, 1.0])
T_real   = np.array([T_mrt, T0, T1])
T_model  = T(t_check)
labels_c = [f"{hora_muerte.strftime('%H:%M')}", "12:30", "13:30"]
colors_c = [COLOR_MORT, COLOR_PTS, COLOR_PTS]

bars = ax3.bar(labels_c, T_real - T_model, color=colors_c, alpha=0.8, width=0.4)
ax3.axhline(0, color=COLOR_TEXT, linewidth=0.8)
ax3.set_ylabel("Error (°C)", fontsize=9)
ax3.set_title("Residuos: T_real − T_modelo", fontsize=10)

for bar, tr, tm in zip(bars, T_real, T_model):
    ax3.text(bar.get_x() + bar.get_width() / 2,
             bar.get_height() + 0.002,
             f"T={tr}°C\nM={tm:.3f}°C",
             ha="center", va="bottom", fontsize=7.5, color=COLOR_TEXT)

# ── TEXTO RESUMEN ─────────────────────────────────────────────
summary = (f"C = {C:.2f}   |   k = {k:.4f} h⁻¹   |   "
           f"t_muerte = {t_mrt:.4f} h   →   "
           f"Hora de muerte ≈ {hora_muerte.strftime('%I:%M %p')}")
fig.text(0.5, 0.01, summary, ha="center", fontsize=9.5,
         color=COLOR_PTS, fontstyle="italic")

plt.suptitle("Enfriamiento de Newton — Taller de Cálculo",
             fontsize=15, fontweight="bold", color=COLOR_TEXT, y=0.98)

plt.savefig("enfriamiento_newton.png",
            dpi=160, bbox_inches="tight", facecolor=fig.get_facecolor())
plt.show()
print("\n[✓] Gráfica guardada como 'enfriamiento_newton.png'")
