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


## INGENIERIA DE DATOS

Separar el dataset en activos/eliminados:

```python
# Separar los datos en usuarios activos y eliminados
usuarios_activos = data[data['user_id'].notna()]
usuarios_eliminados = data[data['deleted_account_id'].notna()]

# Guardar los datos en archivos CSV separados
usuarios_activos.to_csv('drive/MyDrive/ColabNotebooks/Business_Payments/usuarios_activos.csv', index=False)
usuarios_eliminados.to_csv('drive/MyDrive/ColabNotebooks/Business_Payments/usuarios_eliminados.csv', index=False)
```

```python
# Eliminar columnas no necesarias
usuarios_activos = usuarios_activos.drop(columns=['deleted_account_id'], errors='ignore')
usuarios_eliminados = usuarios_eliminados.drop(columns=['user_id'], errors='ignore')
```
Convertir fecha a misma zona horaria:
```python
# Convertir a datetime si aún no lo es
usuarios_activos[fecha_column_activos] = pd.to_datetime(usuarios_activos[fecha_column_activos], errors="coerce")
usuarios_eliminados[fecha_column_eliminados] = pd.to_datetime(usuarios_eliminados[fecha_column_eliminados], errors="coerce")

# Verificar si ya tienen zona horaria asignada y asignar si es necesario
timezone_origen = "Europe/Madrid"  # Cambia esto si es otro timezone

if usuarios_activos[fecha_column_activos].dt.tz is None:
    usuarios_activos[fecha_column_activos] = usuarios_activos[fecha_column_activos].dt.tz_localize(timezone_origen)

if usuarios_eliminados[fecha_column_eliminados].dt.tz is None:
    usuarios_eliminados[fecha_column_eliminados] = usuarios_eliminados[fecha_column_eliminados].dt.tz_localize(timezone_origen)

# Convertir a UTC
usuarios_activos[fecha_column_activos] = usuarios_activos[fecha_column_activos].dt.tz_convert("UTC")
usuarios_eliminados[fecha_column_eliminados] = usuarios_eliminados[fecha_column_eliminados].dt.tz_convert("UTC")
```

Crear columna total solicitudes: 

```python
# Contar solicitudes por usuario en usuarios activos
solicitudes_activos = usuarios_activos["user_id"].value_counts().reset_index()
solicitudes_activos.columns = ["user_id", "total_solicitudes_usuario"]

# Unir la información al dataset original
usuarios_activos = usuarios_activos.merge(solicitudes_activos, on="user_id", how="left")

# Contar solicitudes por usuario en usuarios eliminados
solicitudes_eliminados = usuarios_eliminados["deleted_account_id"].value_counts().reset_index()
solicitudes_eliminados.columns = ["deleted_account_id", "total_solicitudes_usuario"]

# Unir la información al dataset original
usuarios_eliminados = usuarios_eliminados.merge(solicitudes_eliminados, on="deleted_account_id", how="left")
```


Total solicitudes canceladas:

```python
import pandas as pd

# Contar operaciones canceladas o rechazadas en usuarios activos
canceladas_rechazadas_activos = usuarios_activos[
    usuarios_activos["status_cash_request"].isin(["cancelled", "rejected"])
]["user_id"].value_counts().reset_index()
canceladas_rechazadas_activos.columns = ["user_id", "total_operaciones_canceladas_rechazadas"]

# Unir la información al dataset original
usuarios_activos = usuarios_activos.merge(canceladas_rechazadas_activos, on="user_id", how="left")
usuarios_activos["total_operaciones_canceladas_rechazadas"].fillna(0, inplace=True)

# Contar operaciones canceladas o rechazadas en usuarios eliminados
canceladas_rechazadas_eliminados = usuarios_eliminados[
    usuarios_eliminados["status_cash_request"].isin(["cancelled", "rejected"])
]["deleted_account_id"].value_counts().reset_index()
canceladas_rechazadas_eliminados.columns = ["deleted_account_id", "total_operaciones_canceladas_rechazadas"]

# Unir la información al dataset original
usuarios_eliminados = usuarios_eliminados.merge(canceladas_rechazadas_eliminados, on="deleted_account_id", how="left")
usuarios_eliminados["total_operaciones_canceladas_rechazadas"].fillna(0, inplace=True)

# Verificar resultado
print(usuarios_activos[["user_id", "total_operaciones_canceladas_rechazadas"]].head())
print(usuarios_eliminados[["deleted_account_id", "total_operaciones_canceladas_rechazadas"]].head())
```

Crear columnas diferentes valores de fecha:

Mes:

```python
# Convertir a zona horaria UTC
usuarios_activos[fecha_column_activos] = pd.to_datetime(usuarios_activos[fecha_column_activos], utc=True, errors="coerce")
usuarios_eliminados[fecha_column_eliminados] = pd.to_datetime(usuarios_eliminados[fecha_column_eliminados], utc=True, errors="coerce")

# Extraer el mes en formato YYYY-mm
usuarios_activos["mes_solicitud"] = usuarios_activos[fecha_column_activos].dt.strftime("%Y-%m")
usuarios_eliminados["mes_solicitud"] = usuarios_eliminados[fecha_column_eliminados].dt.strftime("%Y-%m")

# Verificar resultado
print(usuarios_activos[["mes_solicitud"]].head())
print(usuarios_eliminados[["mes_solicitud"]].head())
```
<img width="129" alt="image" src="https://github.com/user-attachments/assets/b6369b25-13ef-43b1-98d1-382d92e3ae58" />

Semana:

```python
# Extraer la semana, el nombre del mes y el año
usuarios_activos["semana_solicitud"] = usuarios_activos[fecha_column_activos].dt.isocalendar().week.astype(str) + "_" + \
                                       usuarios_activos[fecha_column_activos].dt.strftime("%B") + "_" + \
                                       usuarios_activos[fecha_column_activos].dt.strftime("%Y")

usuarios_eliminados["semana_solicitud"] = usuarios_eliminados[fecha_column_eliminados].dt.isocalendar().week.astype(str) + "_" + \
                                          usuarios_eliminados[fecha_column_eliminados].dt.strftime("%B") + "_" + \
                                          usuarios_eliminados[fecha_column_eliminados].dt.strftime("%Y")
```

<img width="154" alt="image" src="https://github.com/user-attachments/assets/3cd836fd-5905-4478-ae48-e9b353c1c47f" />

Dia:

```python
# Extraer el día de la semana, la semana del año, el nombre del mes y el año
usuarios_activos["dia_semana_solicitud"] = usuarios_activos[fecha_column_activos].dt.strftime("%A") + "_" + \
                                           usuarios_activos[fecha_column_activos].dt.isocalendar().week.astype(str) + "_" + \
                                           usuarios_activos[fecha_column_activos].dt.strftime("%B") + "_" + \
                                           usuarios_activos[fecha_column_activos].dt.strftime("%Y")

usuarios_eliminados["dia_semana_solicitud"] = usuarios_eliminados[fecha_column_eliminados].dt.strftime("%A") + "_" + \
                                              usuarios_eliminados[fecha_column_eliminados].dt.isocalendar().week.astype(str) + "_" + \
                                              usuarios_eliminados[fecha_column_eliminados].dt.strftime("%B") + "_" + \
                                              usuarios_eliminados[fecha_column_eliminados].dt.strftime("%Y")
```

<img width="218" alt="image" src="https://github.com/user-attachments/assets/bd5a0f67-b7d5-49cd-9ab2-9e58b408dc25" />

Hora:

```python
# Extraer la hora, día de la semana, la semana del año, el nombre del mes y el año
usuarios_activos["hora_solicitud"] = usuarios_activos[fecha_column_activos].dt.hour.astype(str) + "_" + \
                                     usuarios_activos[fecha_column_activos].dt.strftime("%A") + "_" + \
                                     usuarios_activos[fecha_column_activos].dt.isocalendar().week.astype(str) + "_" + \
                                     usuarios_activos[fecha_column_activos].dt.strftime("%B") + "_" + \
                                     usuarios_activos[fecha_column_activos].dt.strftime("%Y")

usuarios_eliminados["hora_solicitud"] = usuarios_eliminados[fecha_column_eliminados].dt.hour.astype(str) + "_" + \
                                        usuarios_eliminados[fecha_column_eliminados].dt.strftime("%A") + "_" + \
                                        usuarios_eliminados[fecha_column_eliminados].dt.isocalendar().week.astype(str) + "_" + \
                                        usuarios_eliminados[fecha_column_eliminados].dt.strftime("%B") + "_" + \
                                        usuarios_eliminados[fecha_column_eliminados].dt.strftime("%Y")
```

<img width="238" alt="image" src="https://github.com/user-attachments/assets/74dd8249-9af7-4275-bd5a-8c3918d30a6b" />

Tasa Rechazo:

```python
# Evitar división por cero
usuarios_activos["tasa_rechazo"] = usuarios_activos["total_operaciones_canceladas_rechazadas"] / usuarios_activos["total_solicitudes_usuario"]
usuarios_eliminados["tasa_rechazo"] = usuarios_eliminados["total_operaciones_canceladas_rechazadas"] / usuarios_eliminados["total_solicitudes_usuario"]
```

<img width="111" alt="image" src="https://github.com/user-attachments/assets/2a3e102b-c316-4dd2-b22c-20531d80c75d" />

Pago tardio ratio:

```python
# Calcular la diferencia en días entre pago y creación
usuarios_activos["dias_para_pago"] = (usuarios_activos[pago_column_activos] - usuarios_activos[fecha_column_activos]).dt.days
usuarios_eliminados["dias_para_pago"] = (usuarios_eliminados[pago_column_eliminados] - usuarios_eliminados[fecha_column_eliminados]).dt.days

# Definir pagos tardíos (más de 30 días)
usuarios_activos["pago_tardio"] = (usuarios_activos["dias_para_pago"] > 30).astype(int)
usuarios_eliminados["pago_tardio"] = (usuarios_eliminados["dias_para_pago"] > 30).astype(int)
```

<img width="158" alt="image" src="https://github.com/user-attachments/assets/cfc4d45e-18b1-4a40-b54a-6f5771faa065" />

Solicitudes Modificadas:

```python
# Contar cuántas veces se ha modificado una solicitud
usuarios_activos["solicitudes_modificadas"] = usuarios_activos.groupby("user_id")[updated_column_activos].transform("count") - 1
usuarios_eliminados["solicitudes_modificadas"] = usuarios_eliminados.groupby("deleted_account_id")[updated_column_eliminados].transform("count") - 1
```

<img width="214" alt="image" src="https://github.com/user-attachments/assets/0f7d8ed2-2531-42b1-b6ee-55a248aa2e05" />


## Graficaciones nuevas variables Activos/Eliminados:

## Histogramas:

<img width="1182" alt="image" src="https://github.com/user-attachments/assets/30c91e4b-8e90-4380-98e8-99ab265e2630" />

<img width="1183" alt="image" src="https://github.com/user-attachments/assets/6e47663d-2d4b-4cfc-a436-caba91713ab2" />

<img width="1203" alt="image" src="https://github.com/user-attachments/assets/e092028e-8b76-4109-a4a8-c787d9edc6ba" />

## Boxplot:

<img width="1196" alt="image" src="https://github.com/user-attachments/assets/0742e074-5f3d-4dca-b66b-9933a4e1ed76" />

<img width="1200" alt="image" src="https://github.com/user-attachments/assets/973ba1f4-6a37-458c-a157-308b19fecc60" />

<img width="1179" alt="image" src="https://github.com/user-attachments/assets/722754ad-a045-478d-94c4-a52505ab0717" />

<img width="1187" alt="image" src="https://github.com/user-attachments/assets/e42b36c2-8f91-42e9-ae63-d3abefd2e141" />

<img width="1191" alt="image" src="https://github.com/user-attachments/assets/95fb8c1e-383b-437d-ac5b-c83a20701a13" />

<img width="1229" alt="image" src="https://github.com/user-attachments/assets/9220510f-b36e-47e2-afdb-468a3d62d673" />

<img width="1203" alt="image" src="https://github.com/user-attachments/assets/93554d74-02ae-42c4-b04e-51990865bd42" />

<img width="1209" alt="image" src="https://github.com/user-attachments/assets/04c2e750-6034-4456-8165-bdf453599ff3" />

## Violinplot

<img width="1180" alt="image" src="https://github.com/user-attachments/assets/107ae1e6-2bfd-46b9-949d-026155c6a9dc" />

<img width="1196" alt="image" src="https://github.com/user-attachments/assets/b6abd909-f06c-4ddc-82c6-a6df5de4baf1" />

<img width="1189" alt="image" src="https://github.com/user-attachments/assets/454a79cf-db82-4bd3-81cd-5ddd47dc3045" />

<img width="1186" alt="image" src="https://github.com/user-attachments/assets/ac009d66-de54-4337-9d3f-06038b020a22" />

<img width="1186" alt="image" src="https://github.com/user-attachments/assets/c7a1a839-228b-4ced-845d-02622ee9d21d" />

<img width="1233" alt="image" src="https://github.com/user-attachments/assets/fee8a358-f6ce-4081-a8ce-2514ab86c754" />

<img width="1188" alt="image" src="https://github.com/user-attachments/assets/8e6b7f45-704a-4416-adfa-c108bb8742cd" />

<img width="1221" alt="image" src="https://github.com/user-attachments/assets/1e996af4-c463-4478-8d49-88f60fdfd390" />

## Dispersión con tendencia

<img width="719" alt="image" src="https://github.com/user-attachments/assets/7bd81cba-ae27-47dd-af24-244c87ad2b44" />

<img width="849" alt="image" src="https://github.com/user-attachments/assets/e2eb6f27-66c0-417f-a8cb-de8a6c899fc4" />

<img width="676" alt="image" src="https://github.com/user-attachments/assets/9f7d1db5-abf5-4e5f-ae03-527edb227d4b" />

<img width="750" alt="image" src="https://github.com/user-attachments/assets/8975ce8b-9444-497a-b5bf-6d2e6469a258" />

<img width="643" alt="image" src="https://github.com/user-attachments/assets/07b2c4c2-3488-44a5-9818-fd31c4f85bc1" />

<img width="616" alt="image" src="https://github.com/user-attachments/assets/773f4d1a-88b2-4119-bc11-2c0615b72c79" />

<img width="651" alt="image" src="https://github.com/user-attachments/assets/e953dadb-c1b6-4f8f-9fe0-e57b2f8fed7a" />

<img width="696" alt="image" src="https://github.com/user-attachments/assets/5c892e78-6240-4fa8-bd2d-03097538c649" />

<img width="835" alt="image" src="https://github.com/user-attachments/assets/f9f07676-9bba-42f9-9a53-b0b657dd5c70" />

<img width="987" alt="image" src="https://github.com/user-attachments/assets/c7d21a4e-c212-43a0-b2b4-b003a43a75f8" />

<img width="753" alt="image" src="https://github.com/user-attachments/assets/03907077-14eb-4c4c-9720-ac18fa56f5e0" />

<img width="882" alt="image" src="https://github.com/user-attachments/assets/664899d2-32de-4070-9fc9-6981b42be501" />

<img width="770" alt="image" src="https://github.com/user-attachments/assets/0f47d52d-58eb-44b8-8d28-b4071b506cbc" />

<img width="721" alt="image" src="https://github.com/user-attachments/assets/0b1adb40-ed73-4983-894c-30401f233818" />

<img width="781" alt="image" src="https://github.com/user-attachments/assets/980374ab-dea0-4ced-b418-77f80c88ebe5" />

<img width="833" alt="image" src="https://github.com/user-attachments/assets/3eabb456-98d5-40da-b52e-781097f0e986" />
