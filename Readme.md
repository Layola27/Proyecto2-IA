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

## EDA

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



```
```python
merged_df.info()

```
![image](https://github.com/user-attachments/assets/03b0dffa-9ff7-413b-8d5e-be616550fbe1)

```python
business_payments.describe()
```

<img width="810" alt="image" src="https://github.com/user-attachments/assets/37711358-ff26-4df7-b009-674b96183d67" />
## Valores faltantes del dataset

Left Join:

<img width="942" alt="image" src="https://github.com/user-attachments/assets/1296d343-ccd9-49db-a3c3-5b2736027a36" />



Activos/Eliminados: 

<img width="1220" alt="image" src="https://github.com/user-attachments/assets/cc1115ca-31e4-4b26-98a1-11112c5d8a81" />



## Valores Categoricos

Join:

<img width="1300" alt="image" src="https://github.com/user-attachments/assets/00b5b172-7825-485a-804e-cf2f1335169a" />

<img width="1300" alt="image" src="https://github.com/user-attachments/assets/f2757268-6514-476b-952c-f826837063a7" />

<img width="1323" alt="image" src="https://github.com/user-attachments/assets/6539b513-08ec-422e-9d42-4d9c3122357a" />

<img width="1267" alt="image" src="https://github.com/user-attachments/assets/19b05959-3af0-44d9-9051-f7b63308f10a" />

<img width="1309" alt="image" src="https://github.com/user-attachments/assets/576d2785-159d-4d5e-b0ed-a3c91e9beb1e" />

<img width="1349" alt="image" src="https://github.com/user-attachments/assets/69aad150-58a6-4d3b-bc0f-384557d7a768" />

<img width="1349" alt="image" src="https://github.com/user-attachments/assets/ce577ee9-8bf7-4c87-a0d4-fe0ae197ca81" />


Activos:

<img width="1227" alt="image" src="https://github.com/user-attachments/assets/07a13b90-aee6-4311-9980-728a474b929c" />

<img width="1057" alt="image" src="https://github.com/user-attachments/assets/d4d92f49-04f0-48a8-986d-74c335508d30" />


Eliminados: 

<img width="1077" alt="image" src="https://github.com/user-attachments/assets/4e8dd6a0-2646-40f0-af42-9bab11a1f961" />

<img width="1209" alt="image" src="https://github.com/user-attachments/assets/b2cf4089-5c37-4ef1-9db4-619bab91dbdb" />

## Boxplot

Join:

<img width="1163" alt="image" src="https://github.com/user-attachments/assets/a7489cdf-e1e6-42e5-8ea2-6dcc58aa378a" />

<img width="1128" alt="image" src="https://github.com/user-attachments/assets/83479ce4-d1ca-4928-97e1-19702dfe15e0" />

Activos/Eliminados:

<img width="803" alt="image" src="https://github.com/user-attachments/assets/a75a452f-41c4-4a87-8f90-5020515e318c" />
<img width="793" alt="image" src="https://github.com/user-attachments/assets/1352f903-950f-4568-918b-bb2c5ba53bda" />

## Violinplot

Join:

<img width="1170" alt="image" src="https://github.com/user-attachments/assets/1d40929f-f6f6-472e-aa32-c63a88087ce4" />

<img width="1136" alt="image" src="https://github.com/user-attachments/assets/4d176a16-9094-4e1c-b339-40d7545e3d6a" />

<img width="1035" alt="image" src="https://github.com/user-attachments/assets/5d182636-a35e-4154-bfd4-5f6fc7876d36" />

<img width="1171" alt="image" src="https://github.com/user-attachments/assets/04eeb90c-9828-44f0-b67e-65d69ec0a897" />

<img width="1193" alt="image" src="https://github.com/user-attachments/assets/38ca9027-1d2d-4182-9b8c-7682fb0caeb2" />

Activos/Eliminados:

<img width="794" alt="image" src="https://github.com/user-attachments/assets/5f0c1798-40e3-412c-ae6e-31f0e71986b1" />

<img width="816" alt="image" src="https://github.com/user-attachments/assets/39ac7358-94aa-4e80-a7da-f0ecdafece5d" />

## Dispersión

<img width="800" alt="image" src="https://github.com/user-attachments/assets/f66c9c5d-16be-4e5e-9b13-fdfda35a3ba0" />

<img width="801" alt="image" src="https://github.com/user-attachments/assets/1a352b7f-e9f8-4514-9e69-02d4606acbc5" />

<img width="789" alt="image" src="https://github.com/user-attachments/assets/6b95510d-dda0-46d6-b01a-d6bdf416e93d" />

<img width="807" alt="image" src="https://github.com/user-attachments/assets/92c79ce8-a82d-4bf1-8c96-310088690ab8" />

<img width="812" alt="image" src="https://github.com/user-attachments/assets/65a0a2f1-9bcf-4429-84c0-c43691a94ac1" />

Con tendencia:

<img width="620" alt="image" src="https://github.com/user-attachments/assets/1a7556d3-c7d5-4e94-81ff-295495e36996" />

<img width="708" alt="image" src="https://github.com/user-attachments/assets/a55f8b38-30a7-4c1c-a6ff-a1b279ea2170" />

## Series Temporales

Join:

<img width="963" alt="image" src="https://github.com/user-attachments/assets/6848d581-d472-4833-b120-e5cac7fb5975" />

<img width="944" alt="image" src="https://github.com/user-attachments/assets/42d762f1-60db-4a8a-a684-01156f877351" />

<img width="946" alt="image" src="https://github.com/user-attachments/assets/23f579ca-0c8c-471d-be3b-d52fe56a6e42" />

<img width="938" alt="image" src="https://github.com/user-attachments/assets/311c9c60-d55b-46e8-bd94-1669012a5abe" />

<img width="938" alt="image" src="https://github.com/user-attachments/assets/6798fd27-309c-4424-afb2-2e68bdbb0c3c" />

<img width="958" alt="image" src="https://github.com/user-attachments/assets/0101e7c8-0f08-4f27-ad7a-27b3691fff67" />

Activos/Eliminados:

<img width="1189" alt="image" src="https://github.com/user-attachments/assets/7b9698ac-9c5c-4ccb-9497-ddc0d3e4e681" />

<img width="1191" alt="image" src="https://github.com/user-attachments/assets/9a653250-9280-4592-9f00-8d5d2eba307f" />

