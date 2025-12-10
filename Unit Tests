
from asce7.wind import SiteWind, si_q

def test_kz_monotonic():
    low = SiteWind(V_mph=115, exposure='C', z_ft=15)
    high= SiteWind(V_mph=115, exposure='C', z_ft=90)
    assert high.Kz() > low.Kz()

def test_q_asd_vs_ult():
    ult = SiteWind(V_mph=115, exposure='C', z_ft=30, method='ULT')
    asd = SiteWind(V_mph=115, exposure='C', z_ft=30, method='ASD')
    assert abs(asd.q() - ult.q()*0.6) < 1e-9

def test_si_helper():
    # ~115 mph, z=30 ft
    assert si_q(V_ms=51.4, z_m=9.14, exposure='C') > 0.0
