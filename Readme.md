# Análisis de Tarifas y Solicitudes de Efectivo en Business Payments

## Introducción
Este proyecto tiene como objetivo analizar y extraer insights clave a partir de datos relacionados con tarifas (`fees`) y solicitudes de adelanto de efectivo (`cash requests`) dentro de Business Payments, una empresa de servicios financieros. A través de este estudio, se busca comprender mejor el comportamiento de los usuarios, evaluar el impacto de diferentes tipos de tarifas y detectar patrones que puedan ayudar en la toma de decisiones estratégicas.

## Conjuntos de Datos
El análisis se basa en tres conjuntos de datos principales:

### 1. Tarifas (`fees`)
Contiene información detallada sobre las tarifas aplicadas a los usuarios por diferentes conceptos, como pagos instantáneos, incidentes de reembolso fallidos y aplazamientos de pago.
- **Número de registros:** 21,061
- **Número de columnas:** 13
- **Principales columnas:**  
  - `id`: Identificador único de la tarifa.
  - `cash_request_id`: Identificador de la solicitud de efectivo relacionada.
  - `type`: Tipo de tarifa (`instant_payment`, `split_payment`, `incident`, `postpone`).
  - `status`: Estado de la tarifa (`confirmed`, `rejected`, `cancelled`, `accepted`).
  - `total_amount`: Monto total de la tarifa.
  - `created_at`: Fecha de creación de la tarifa.

### 2. Solicitudes de Efectivo (`cash requests`)
Registra todas las transacciones en las que los usuarios solicitan adelantos de efectivo, permitiendo analizar tendencias de uso y posibles factores de riesgo.
- **Número de registros:** 23,970
- **Número de columnas:** 16
- **Principales columnas:**  
  - `id`: Identificador único de la solicitud.
  - `amount`: Monto solicitado.
  - `status`: Estado de la solicitud (`approved`, `pending`, `rejected`).
  - `user_id`: Identificador del usuario que realizó la solicitud.
  - `reimbursement_date`: Fecha estimada de reembolso.
  - `transfer_type`: Tipo de transferencia utilizada.

### 3. Lexique (`Lexique - Data Analyst.xlsx`)
Proporciona definiciones y referencias clave para comprender los términos utilizados en los datos.
- **Número de registros:** 16,375
- **Número de columnas:** 2
- **Columnas:**  
  - `Column name`: Nombre del término en la base de datos.
  - `Description`: Descripción del término.

## Objetivos del Análisis
Este estudio busca responder preguntas clave sobre el uso y rendimiento de los servicios financieros de Business Payments. Los principales objetivos incluyen:
- **Exploración y limpieza de datos** para garantizar calidad y consistencia.
- **Identificación de patrones de uso y frecuencia de pagos** para entender el comportamiento de los usuarios.
- **Análisis de cohortes** para entender la retención y el comportamiento de los usuarios a lo largo del tiempo.
- **Modelo de clasificación** para detectar usuarios con possibilidad de irse de la plataforma.

## Tecnologías y Herramientas Utilizadas
- **Python**: Análisis de datos con `pandas`, `numpy`, `matplotlib`, `seaborn`.
- **Google Colab**: Desarrollo y ejecución de código en la nube.
- **Jupyter Notebook**: Exploración de datos interactiva.
- **Excel**: Revisión y validación de datos adicionales.

##EDA

Cargar Archivos:

```python
import pandas as pd
# Cargar los datasets
fees_path = "drive/MyDrive/ColabNotebooks/Business_Payments/extract - fees - data analyst - .csv"
cash_request_path = "drive/MyDrive/ColabNotebooks/Business_Payments/extract - cash request - data analyst.csv"

fees_df = pd.read_csv(fees_path)
cash_request_df = pd.read_csv(cash_request_path)

Join (Left):

```python
import pandas as pd

# Cargar los datasets
fees_path = "drive/MyDrive/ColabNotebooks/Business_Payments/extract - fees - data analyst - .csv"
cash_request_path = "drive/MyDrive/ColabNotebooks/Business_Payments/extract - cash request - data analyst.csv"

fees_df = pd.read_csv(fees_path)
cash_request_df = pd.read_csv(cash_request_path)

# Fusionar los datasets usando cash_request_id para vincularlos
merged_df = cash_request_df.merge(
    fees_df,
    left_on="id",  # En cash_request_df la columna se llama "id"
    right_on="cash_request_id",  # En fees_df la clave es "cash_request_id"
    how="left",  # Mantener todas las filas de cash_request_df
    suffixes=("", "_fees")  # Agregar sufijo "_fees" a las columnas duplicadas
)

# Eliminar la columna cash_request_id de fees después del merge, ya que es redundante
merged_df.drop(columns=["cash_request_id"], inplace=True)

# Mostrar las primeras filas para verificar la fusión
print(merged_df.head())

```python
merged_df.info()





