
# asce7/wind.py
# Core wind calculations per ASCE/SEI 7-16/7-22 (public summaries).
# Units: mph, ft, psf internally; SI helper provided.

from dataclasses import dataclass
from typing import Literal

Exposure = Literal['B','C','D']
Enclosure = Literal['open','enclosed','partially_enclosed','partially_open']
DesignMethod = Literal['ULT','ASD']

# Kz via table-equivalent formula: alpha & zg per ASCE conventions (B/C/D)
EXPOSURE_PARAMS = {
    'B': {'alpha': 7.0,  'zg_ft': 1200.0, 'zmin_ft': 30.0},
    'C': {'alpha': 9.5,  'zg_ft': 900.0,  'zmin_ft': 15.0},
    'D': {'alpha': 11.5, 'zg_ft': 700.0,  'zmin_ft': 7.0},
}

# GCpi (ASCE 7 Table 26.11-1)
GCPI = {
    'open':               (+0.00, -0.00),
    'enclosed':           (+0.18, -0.18),
    'partially_enclosed': (+0.55, -0.55),
    'partially_open':     (+0.18, -0.18),  # default when not strictly “open” or “partially enclosed”
}

KD_DEFAULT = 0.85  # Table 26.6-1 (typical for buildings)

@dataclass
class SiteWind:
    V_mph: float
    exposure: Exposure
    z_ft: float
    Kzt: float = 1.0
    Ke: float = 1.0
    Kd: float = KD_DEFAULT
    method: DesignMethod = 'ULT'  # 'ULT' or 'ASD'

    def Kz(self) -> float:
        p = EXPOSURE_PARAMS[self.exposure]
        z_ft = max(self.z_ft, p['zmin_ft'])
        alpha, zg = p['alpha'], p['zg_ft']
        return 2.01 * (z_ft/zg)**(2.0/alpha)

    def q(self) -> float:
        """
        q (psf) = 0.00256 * Kz * Kzt * Ke * Kd * V^2   (ULT)
        ASD: multiply by 0.6
        """
        base = 0.00256 * self.Kz() * self.Kzt * self.Ke * self.Kd * (self.V_mph**2)
        return base * (0.6 if self.method == 'ASD' else 1.0)

def mwfrs_pressure(q_ext_psf: float, G: float, Cp: float, q_int_psf: float, GCpi: float) -> float:
    # P = q * G * Cp - qi * GCpi
    return q_ext_psf * G * Cp - q_int_psf * GCpi

def cnc_pressure(qh_psf: float, GCp: float, GCpi: float, Kd: float | None = None) -> float:
    # p = qh * [GCp - GCpi]; in ASCE 7-22, Kd may apply at pressure stage → optional
    p = qh_psf * (GCp - GCpi)
    return p if Kd is None else p * Kd

LOW_RISE_ROOF_GCP = {'zone_1p': -0.9, 'zone_1': -1.7, 'zone_2': -2.3, 'zone_3': -3.2}  # typical values

# SI helpers
FT_TO_M = 0.3048; MPH_TO_MS = 0.44704; PSF_TO_KPA = 0.04788
def si_q(V_ms: float, z_m: float, exposure: Exposure, Kzt: float = 1.0, Ke: float = 1.0, Kd: float = KD_DEFAULT, method: DesignMethod = 'ULT') -> float:
    z_ft = z_m / FT_TO_M; V_mph = V_ms / MPH_TO_MS
    q_psf = SiteWind(V_mph=V_mph, exposure=exposure, z_ft=z_ft, Kzt=Kzt, Ke=Ke, Kd=Kd, method=method).q()
    return q_psf * PSF_TO_KPA
