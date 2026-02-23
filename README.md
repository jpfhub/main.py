import streamlit as st
import pandas as pd
from fpdf import FPDF
from io import BytesIO
import datetime

st.set_page_config(
    page_title="🚌 Plataforma FUEC ADSO SENA",
    page_icon="🚌",
    layout="wide"
)

# Base de datos vehículos (RF01)
vehiculos_db = {
    'ABC123': {
        'operador': 'Juan Pablo Florez Aguazaco',
        'servicio': 'Transporte Especial Nacional',
        'origen': 'Bogotá D.C.',
        'destino': 'Medellín',
        'vigencia': '2026-03-31'
    },
    'XYZ456': {
        'operador': 'María Angela López',
        'servicio': 'Especial Cali-Barranquilla',
        'origen': 'Cali',
        'destino': 'Barranquilla',
        'vigencia': '2026-04-30'
    },
    'DEF789': {
        'operador': 'Carlos Ramírez Soto',
        'servicio': 'Nacional Bogotá-Cali',
        'origen': 'Bogotá D.C.',
        'destino': 'Cali',
        'vigencia': '2026-02-28'
    }
}

st.title("🚛 Plataforma Digital FUEC - Generación Automática")
st.markdown("*Proyecto ADSO SENA: Gestión FUEC Transporte Especial [GA1-220501092-AA4-EV01]*")

col1, col2 = st.columns([3, 1])

with col1:
    placa_input = st.text_input(
        "Ingrese placa del vehículo:",
        placeholder="Ejemplo: ABC123",
        help="Busca en BD y genera FUEC prediligenciado (RF02/RF03)"
    ).upper().strip()

with col2:
    if st.button("🔥 Generar FUEC PDF", type="primary", use_container_width=True):
        pass  # Trigger

if placa_input in vehiculos_db:
    datos = vehiculos_db[placa_input]
    
    st.success(f"✅ **Vehículo encontrado**: {placa_input}")
    
    # Muestra datos prediligenciados
    col_a, col_b = st.columns(2)
    with col_a:
        st.metric("Operador", datos['operador'])
        st.metric("Servicio", datos['servicio'])
    with col_b:
        st.metric("Ruta", f"{datos['origen']} → {datos['destino']}")
        st.metric("Vigencia", datos['vigencia'])
    
    # Genera PDF FUEC oficial
    @st.cache_data
    def crear_pdf(placa, datos):
        buffer = BytesIO()
        pdf = FPDF('P', 'mm', 'A4')
        pdf.add_page()
        
        # Header FUEC
        pdf.set_font('Arial', 'B', 20)
        pdf.cell(0, 15, 'FUEC', 0, 1, 'C')
        pdf.set_font('Arial', 'B', 14)
        pdf.cell(0, 10, 'FORMATO ÚNICO EXTRACTO DE CONTRATO', 0, 1, 'C')
        pdf.ln(5)
        
        # Datos
        pdf.set_font('Arial', 'B', 12)
        pdf.cell(50, 8, 'Placa:', 0, 0)
        pdf.set_font('Arial', '', 12)
        pdf.cell(120, 8, placa, 0, 1)
        
        pdf.set_font('Arial', 'B', 12)
        pdf.cell(50, 8, 'Operador:', 0, 0)
        pdf.set_font('Arial', '', 12)
        pdf.cell(120, 8, datos['operador'], 0, 1)
        
        pdf.set_font('Arial', 'B', 12)
        pdf.cell(50, 8, 'Servicio:', 0, 0)
        pdf.set_font('Arial', '', 12)
        pdf.cell(120, 8, datos['servicio'], 0, 1)
        
        pdf.set_font('Arial', 'B', 12)
        pdf.cell(50, 8, 'Ruta:', 0, 0)
        pdf.set_font('Arial', '', 12)
        pdf.cell(120, 8, f"{datos['origen']} - {datos['destino']}", 0, 1)
        
        pdf.set_font('Arial', 'B', 12)
        pdf.cell(50, 8, 'Vigencia:', 0, 0)
        pdf.set_font('Arial', '', 12)
        pdf.cell(120, 8, datos['vigencia'], 0, 1)
        
        pdf.ln(10)
        pdf.set_font('Arial', '', 10)
        pdf.cell(0, 6, f"Generado: {datetime.datetime.now().strftime('%d/%m/%Y %H:%M:%S')} - Plataforma ADSO SENA", 0, 1, 'C')
        
        pdf.output(buffer)
        return buffer.getvalue()
    
    pdf_data = crear_pdf(placa_input, datos)
    
    st.download_button(
        label="📥 **Descargar FUEC Oficial PDF**",
        data=pdf_data,
        file_name=f"FUEC_{placa_input}_{datetime.datetime.now().strftime('%Y%m%d_%H%M')}.pdf",
        mime="application/pdf",
        use_container_width=True
    )
    
    st.balloons()
    
else:
    if placa_input:
        st.error("❌ **Placa no registrada**. Agregue vehículos en próxima fase (RF01).")

# Footer
st.markdown("---")
st.markdown("""
*👨‍💻 Desarrollado por Juan Pablo Florez - Tecnólogo ADSO SENA 2026*  
*📄 Evidencia: Especificaciones funcionales Plataforma FUEC[file:31]*
""")

if st.button("🎉 Demo Exitosa - Listo para Empresa!"):
    st.success("¡Tu FUEC generator funciona! Comparte esta URL.")
