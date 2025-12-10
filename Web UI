
import streamlit as st, pandas as pd
from asce7.wind import SiteWind, mwfrs_pressure, cnc_pressure, LOW_RISE_ROOF_GCP, GCPI, KD_DEFAULT, PSF_TO_KPA
from asce7.exposure import EXPOSURE_HELP
from asce7.enclosure import Openings

st.set_page_config(page_title='ASCE 7 Wind Load Calculator', layout='wide')
st.title('ASCE 7 Wind Load Calculator (MWFRS & C&C)')
st.caption('ASCE 7-16 / 7-22 • visual, unit-safe, and auditable')

with st.sidebar:
    st.header('Project & Site')
    address = st.text_input('Project address (for your report)', '')
    risk    = st.selectbox('Risk Category', ['I','II','III','IV'], index=1)
    V       = st.number_input('Basic wind speed V (mph)', min_value=60.0, max_value=250.0, value=115.0, step=1.0)
    exposure= st.selectbox('Exposure (B/C/D)', ['B','C','D'], index=1, help=EXPOSURE_HELP['C'])
    z       = st.number_input('Reference height z (ft)', min_value=10.0, max_value=1000.0, value=30.0, step=1.0)
    Kzt     = st.number_input('Topographic factor Kzt', min_value=1.0, max_value=3.0, value=1.0, step=0.05)
    Ke      = st.number_input('Ground elevation factor Ke', min_value=0.7, max_value=1.0, value=1.0, step=0.01)
    Kd      = st.number_input('Directionality factor Kd', min_value=0.7, max_value=1.0, value=KD_DEFAULT, step=0.01)
    method  = st.radio('Design method', ['ULT','ASD'], index=0)
    G       = st.number_input('Gust factor G (rigid default 0.85)', min_value=0.5, max_value=1.5, value=0.85, step=0.01)

site = SiteWind(V_mph=V, exposure=exposure, z_ft=z, Kzt=Kzt, Ke=Ke, Kd=Kd, method=method)
qz   = site.q()
qh   = SiteWind(V_mph=V, exposure=exposure, z_ft=z, Kzt=Kzt, Ke=Ke, Kd=Kd, method=method).q()

st.subheader('Velocity pressure')
c1,c2,c3 = st.columns(3)
c1.metric('Kz', f"{site.Kz():.3f}")
c2.metric('qz (psf)', f"{qz:.2f}")
c3.metric('qh (psf)', f"{qh:.2f}")

st.divider()
st.subheader('Enclosure classification & GCpi')
with st.expander('Enter opening areas (sq ft) or skip if known'):
    openings = Openings(
        gross_N=st.number_input('Gross area North wall', value=1000.0), open_N=st.number_input('Openings North wall', value=10.0),
        gross_E=st.number_input('Gross area East wall',  value=1000.0), open_E=st.number_input('Openings East wall',  value=10.0),
        gross_S=st.number_input('Gross area South wall', value=1000.0), open_S=st.number_input('Openings South wall', value=10.0),
        gross_W=st.number_input('Gross area West wall',  value=1000.0), open_W=st.number_input('Openings West wall',  value=10.0),
        gross_Roof=st.number_input('Gross area Roof',     value=2000.0), open_Roof=st.number_input('Openings Roof',     value=0.0),
    )
    auto_enclosure, msg = openings.classify()
    st.info(f"Auto classification: {auto_enclosure} – {msg}")

enclosure = st.selectbox('Final enclosure (override allowed)',
                         ['open','enclosed','partially_enclosed','partially_open'],
                         index=['open','enclosed','partially_enclosed','partially_open'].index(auto_enclosure))
GCpi_pos, GCpi_neg = GCPI[enclosure]
st.write(f"GCpi: +{GCpi_pos:.2f} / {GCpi_neg:.2f}")

st.divider()
st.subheader('MWFRS design pressures (example)')
Cp_windward = st.number_input('Cp windward wall', value=0.8)
Cp_leeward  = st.number_input('Cp leeward wall (negative)', value=-0.5)
p_windward  = mwfrs_pressure(q_ext_psf=qz, G=G, Cp=Cp_windward, q_int_psf=qz, GCpi=GCpi_pos)
p_leeward   = mwfrs_pressure(q_ext_psf=qh, G=G, Cp=Cp_leeward,  q_int_psf=qh, GCpi=GCpi_neg)
mwfrs_df    = pd.DataFrame({
    'Surface': ['Windward wall','Leeward wall'],
    'q (psf)': [qz, qh],
    'Cp':      [Cp_windward, Cp_leeward],
    'GCpi used':[GCpi_pos, GCpi_neg],
    'P (psf)': [p_windward, p_leeward],
    'P (kPa)': [p_windward*PSF_TO_KPA, p_leeward*PSF_TO_KPA]
})
st.dataframe(mwfrs_df, use_container_width=True)

st.divider()
st.subheader('C&C – low-rise roof (typical zones)')
cols = st.columns(4)
for i,(zone, val) in enumerate(LOW_RISE_ROOF_GCP.items()):
    cols[i].number_input(f"GCp {zone}", value=val, key=f"gcp_{zone}")
rows = []
for zone in LOW_RISE_ROOF_GCP:
    GCp = st.session_state[f"gcp_{zone}"]
    rows.append({'Zone': zone, 'GCp': GCp, 'GCpi': GCpi_neg, 'qh (psf)': qh,
                 'p (psf)': cnc_pressure(qh_psf=qh, GCp=GCp, GCpi=GCpi_neg),
                 'p (kPa)': cnc_pressure(qh_psf=qh, GCp=GCp, GCpi=GCpi_neg)*PSF_TO_KPA})
st.dataframe(pd.DataFrame(rows), use_container_width=True)

st.divider()
st.subheader('Import spreadsheet to pre-fill geometry (optional)')
up = st.file_uploader('Upload Excel (.xlsx) to parse height & widths (Basildon example)', type=['xlsx'])
if up is not None:
    try:
        xls = pd.ExcelFile(up)
        df  = pd.read_excel(xls, sheet_name=xls.sheet_names[0])
        # naive discovery of “Building height” row
        height_rows = df.apply(lambda col: col.astype(str).str.contains('Building height', case=False, na=False)).any(axis=1)
        rows = df.loc[height_rows]
        if not rows.empty:
            num = pd.to_numeric(rows.select_dtypes(include=['number']).stack(), errors='coerce')
            if not num.empty:
                st.success(f"Detected building height candidate: {float(num.iloc[0]):.2f} (units as in spreadsheet)")
        st.write('Preview:', df.head())
    except Exception as e:
        st.error(f"Failed to parse Excel: {e}")

st.divider()
st.subheader('Export report')
csv = pd.concat([mwfrs_df.assign(Category='MWFRS'), pd.DataFrame(rows).assign(Category='C&C')]).to_csv(index=False)
st.download_button('Download CSV report', csv, file_name='asce7_wind_report.csv', mime='text/csv')
st.caption('Always verify exposure, enclosure, and coefficients against the adopted ASCE 7 edition in your jurisdiction.')
