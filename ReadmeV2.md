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

## DATASET

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

```

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

## Correlación:

<img width="921" alt="image" src="https://github.com/user-attachments/assets/0999e2bf-7183-4a51-a620-5f27701c488f" />

<img width="590" alt="image" src="https://github.com/user-attachments/assets/d9834953-2ac5-452c-a8ce-dee14894efb2" />

<img width="930" alt="image" src="https://github.com/user-attachments/assets/caea4451-4aae-40c1-b3f2-8cb645459383" />

<img width="555" alt="image" src="https://github.com/user-attachments/assets/066bd852-2d56-41a5-99e4-6a495a126fd4" />

Validar la causalidad entre variables con modelo de regresión multiple (statsmodels):

```python
def analizar_causalidad(data, var_x, var_y):
    # Seleccionar solo columnas numéricas
    data = data.select_dtypes(include=['number'])

    # Verificar si existen otras variables que puedan ser confusoras
    confusores = [col for col in data.columns if col not in [var_x, var_y]]

    if not confusores:
        print(f"\nNo hay confusores disponibles para la relación {var_x} -> {var_y}. Se realizará una regresión simple.")
        X = data[[var_x]]
    else:
        print(f"\nEvaluando causalidad de {var_x} sobre {var_y} controlando por {confusores}.")
        X = data[[var_x] + confusores]

    y = data[var_y]

    # Eliminar filas con valores NaN o infinitos
    df = pd.concat([X, y], axis=1).dropna().replace([np.inf, -np.inf], np.nan).dropna()

    # Separar X e y después de limpiar datos
    X = df.drop(columns=[var_y])
    y = df[var_y]

    # Agregar una constante para la regresión
    X = sm.add_constant(X)

    # Ajustar el modelo de regresión
    modelo = sm.OLS(y, X).fit()

    return modelo.summary()
```
<img width="756" alt="image" src="https://github.com/user-attachments/assets/a58fa434-ceea-4914-b698-79e7c2bfaf71" />

<img width="723" alt="image" src="https://github.com/user-attachments/assets/50c4a3b9-05f5-4b78-a44f-68c2c363049d" />

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

<img width="569" alt="image" src="https://github.com/user-attachments/assets/3e127d48-e88b-476c-b14a-c353ecf52a01" />

<img width="677" alt="image" src="https://github.com/user-attachments/assets/6b137f19-5e0e-4910-ae77-ffb0acb19ff5" />

<img width="611" alt="image" src="https://github.com/user-attachments/assets/71bc4591-10cf-4114-b8a7-b26d6d0b4efc" />

<img width="606" alt="image" src="https://github.com/user-attachments/assets/5fc8bf6b-af38-44ad-bbff-61417fa2415b" />

<img width="661" alt="image" src="https://github.com/user-attachments/assets/578c859e-dfa7-40db-80b4-a0f588afee68" />

<img width="591" alt="image" src="https://github.com/user-attachments/assets/a0265ba0-c1f4-4f57-a9b1-9cae487a2f92" />

<img width="577" alt="image" src="https://github.com/user-attachments/assets/8953662d-8015-4998-ba91-4a2f06083d96" />

<img width="649" alt="image" src="https://github.com/user-attachments/assets/057ac1f5-339c-411d-b9c0-0dbd19c6c055" />

<img width="776" alt="image" src="https://github.com/user-attachments/assets/5c9050c8-cb7c-42b4-8f1c-e9d30c160c8a" />

<img width="895" alt="image" src="https://github.com/user-attachments/assets/fbbdf9b9-0c61-4455-8c4e-ba5e9ec3eae0" />

<img width="812" alt="image" src="https://github.com/user-attachments/assets/062f37fd-f7ed-4a71-acae-5e31f643e297" />

<img width="708" alt="image" src="https://github.com/user-attachments/assets/143ce5f1-1a9f-4c4f-bae2-b7ae6992e1cf" />

<img width="726" alt="image" src="https://github.com/user-attachments/assets/592cdbfc-813a-49d5-b1c1-66816406a588" />

<img width="763" alt="image" src="https://github.com/user-attachments/assets/15e2a5eb-05b0-4cf0-a009-895cb8dc24ee" />

<img width="693" alt="image" src="https://github.com/user-attachments/assets/0b6eee3a-eb11-4722-b4f9-e4b34ce70b24" />

<img width="689" alt="image" src="https://github.com/user-attachments/assets/cef338fb-0c66-4990-a57f-1859e4efdf4c" />

## Series temporales amount y num. de transacciones

<img width="560" alt="image" src="https://github.com/user-attachments/assets/b830f0b8-ba4e-4a10-97b6-01ad9618fffd" />

<img width="676" alt="image" src="https://github.com/user-attachments/assets/7df10922-47c3-4eb2-a42b-a0f7c4bf81e9" />

<img width="785" alt="image" src="https://github.com/user-attachments/assets/adfb2e6d-2402-4548-a005-1ee1819c1cab" />

<img width="593" alt="image" src="https://github.com/user-attachments/assets/0ef187a0-5f60-4185-bfbd-a5997483709c" />

## Analisis de relevacia de fechas

<img width="876" alt="image" src="https://github.com/user-attachments/assets/0189f66a-cdb7-4948-8b29-40c3f2712fb1" />

<img width="867" alt="image" src="https://github.com/user-attachments/assets/d0223141-07b5-4b5b-90a6-38a404760aa3" />

<img width="874" alt="image" src="https://github.com/user-attachments/assets/cc70df38-5818-4f5b-8004-a4d777544c8d" />

<img width="854" alt="image" src="https://github.com/user-attachments/assets/37ff22b8-b1eb-46c1-8043-09ff863d3b8d" />

<img width="1022" alt="image" src="https://github.com/user-attachments/assets/0723b699-265d-4637-a08b-39288cc68569" />

<img width="1016" alt="image" src="https://github.com/user-attachments/assets/0e8ff6f8-3565-4013-8267-f3f406249a36" />

<img width="1019" alt="image" src="https://github.com/user-attachments/assets/4cc2894a-8902-467a-8387-898de5cf5f17" />

<img width="1008" alt="image" src="https://github.com/user-attachments/assets/ccc2f0ec-c3dd-4926-8714-c39c94e5a8ae" />

## Analisis cohorte primer solicitud

```python
usuarios_activos["cohorte_primer_solicitud"] = usuarios_activos.groupby("user_id")["created_at"].transform("min").dt.to_period("M")
usuarios_eliminados["cohorte_primer_solicitud"] = usuarios_eliminados.groupby("deleted_account_id")["created_at"].transform("min").dt.to_period("M")
```

<img width="882" alt="image" src="https://github.com/user-attachments/assets/262c706a-3483-462b-b251-ef59514c04d6" />

<img width="871" alt="image" src="https://github.com/user-attachments/assets/ad7376eb-cd86-4052-bcf2-e75aa3d2021a" />

<img width="851" alt="image" src="https://github.com/user-attachments/assets/9f86fb11-d05d-4c35-bcbd-1c5dab956ea2" />

<img width="830" alt="image" src="https://github.com/user-attachments/assets/618862f2-cd64-4e3f-b66d-c93ffe7a3013" />

<img width="879" alt="image" src="https://github.com/user-attachments/assets/eea59135-0587-49cb-b561-dc52aa272597" />

<img width="868" alt="image" src="https://github.com/user-attachments/assets/4fbe4c38-69ec-4332-a3ca-4a2ce54cce7e" />

<img width="819" alt="image" src="https://github.com/user-attachments/assets/cc843ef2-439e-47de-88eb-568716689cfd" />

<img width="831" alt="image" src="https://github.com/user-attachments/assets/5dcac64b-9873-426d-911e-bc92bd3548ea" />


## Analisis cohorte vida del usuario

```python
import pandas as pd

# Calcular el tiempo de vida del usuario (días entre primera y última solicitud)
usuarios_activos["tiempo_vida"] = (usuarios_activos.groupby("user_id")["created_at"].transform("max") - 
                                   usuarios_activos.groupby("user_id")["created_at"].transform("min")).dt.days

usuarios_eliminados["tiempo_vida"] = (usuarios_eliminados.groupby("deleted_account_id")["created_at"].transform("max") - 
                                      usuarios_eliminados.groupby("deleted_account_id")["created_at"].transform("min")).dt.days

# Crear cohortes basados en el tiempo de vida
def clasificar_tiempo_vida(dias):
    if dias < 30:
        return "0-30 días"
    elif dias < 90:
        return "1-3 meses"
    elif dias < 180:
        return "3-6 meses"
    elif dias < 365:
        return "6-12 meses"
    else:
        return "Más de 1 año"

usuarios_activos["cohorte_tiempo_vida"] = usuarios_activos["tiempo_vida"].apply(clasificar_tiempo_vida)
usuarios_eliminados["cohorte_tiempo_vida"] = usuarios_eliminados["tiempo_vida"].apply(clasificar_tiempo_vida)
```

<img width="884" alt="image" src="https://github.com/user-attachments/assets/11a17d88-547a-49ba-b72e-d5dd1b701936" />

<img width="871" alt="image" src="https://github.com/user-attachments/assets/1203a65d-5303-41b2-932f-493ab6a8f48a" />

<img width="921" alt="image" src="https://github.com/user-attachments/assets/971ad81a-ecaa-44ce-ad09-cd4275eb44ad" />

<img width="877" alt="image" src="https://github.com/user-attachments/assets/f02702df-726f-49ec-9212-17e5938ca2fa" />

## Modelos:

## Modelo de regresión para "adelanto de efectivo por cohorte(primer solicitud) y mes":

Matriz de valores: 

<img width="908" alt="image" src="https://github.com/user-attachments/assets/07b3fcb1-264b-41f4-9eaf-e3c726eb4601" />

Modelos Iniciáles:

<img width="1387" alt="image" src="https://github.com/user-attachments/assets/55967674-a4e0-48bd-8fcd-8a2637e60f45" />

<img width="1380" alt="image" src="https://github.com/user-attachments/assets/77eda97f-3997-4b19-8dd1-390bb10b5d57" />

<img width="1382" alt="image" src="https://github.com/user-attachments/assets/9ecb85d4-8564-4047-b342-210a296ea719" />

Modelo ajustado:

```python
# 1️⃣ Aplicar la transformación logarítmica en 'amount'
cohorte_2019_12_log = np.log1p(cohort_revenue.loc['2019-12'].dropna())

# 3️⃣ Separar 80% interpolación y 20% extrapolación
split_idx = int(len(X) * 0.80)
X_interp, X_extrap = X_numeric[:split_idx], X_numeric[split_idx:]
y_interp, y_extrap = y[:split_idx], y[split_idx:]

# 4️⃣ Dentro del 80% de interpolación, dividir en 80% entrenamiento y 20% prueba
X_train, X_test, y_train, y_test = train_test_split(X_interp, y_interp, test_size=0.20, random_state=42)

# 5️⃣ Ajustar modelo de regresión polinómica con Ridge para evitar sobreajuste
degree = 4  # Se puede probar con otros grados 
alpha = 0.0072  # Regularización

poly = PolynomialFeatures(degree)
X_train_poly = poly.fit_transform(X_train)
X_test_poly = poly.transform(X_test)
X_interp_poly = poly.transform(X_interp)
X_extrap_poly = poly.transform(X_extrap)

# 6️⃣ Entrenar modelo con Ridge
model = Ridge(alpha=alpha)
model.fit(X_train_poly, y_train)
```

<img width="899" alt="image" src="https://github.com/user-attachments/assets/4f1722a3-6e9b-490e-9646-b2108999c31b" />

<img width="788" alt="image" src="https://github.com/user-attachments/assets/e3797bce-b9c2-4523-b2c7-d4b3679ae4b2" />

## Modelo de regresión para "adelanto de efectivo por cohorte(primer solicitud) y semana":

Matriz de valores:

<img width="840" alt="image" src="https://github.com/user-attachments/assets/8ba236e8-4da4-45a7-ad58-00d8780e9cf7" />

Modelos iniciales:

<img width="1402" alt="image" src="https://github.com/user-attachments/assets/c16d95e0-48fb-4413-8a1f-e3e1818e8b11" />

<img width="1398" alt="image" src="https://github.com/user-attachments/assets/f7859bce-3b09-4522-847a-956ba9102ca7" />

<img width="1385" alt="image" src="https://github.com/user-attachments/assets/d1ba5637-f35f-4c02-9257-ebbda0efcf44" />

Busquéda mejores parametros para modelo:

```python
# Definir rango de grados de polinomio a probar
degrees = range(1, 16)

# Almacenar resultados
results = []

# Iterar sobre cada grado de polinomio
for degree in degrees:
    # Transformación polinómica
    poly = PolynomialFeatures(degree)
    X_train_poly = poly.fit_transform(X_train)
    X_test_poly = poly.transform(X_test)

    # Modelo Ridge con alpha fijo
    alpha = 0.01  # Se puede ajustar si es necesario
    model = Ridge(alpha=alpha)
    model.fit(X_train_poly, y_train)

    # Predicciones
    y_train_pred = model.predict(X_train_poly)
    y_test_pred = model.predict(X_test_poly)

    # Revertir transformación logarítmica
    y_train_pred_orig = np.expm1(y_train_pred)
    y_test_pred_orig = np.expm1(y_test_pred)
    y_train_orig = np.expm1(y_train)
    y_test_orig = np.expm1(y_test)

    # Calcular métricas de error
    rmse_train = np.sqrt(mean_squared_error(y_train_orig, y_train_pred_orig))
    rmse_test = np.sqrt(mean_squared_error(y_test_orig, y_test_pred_orig))
    r2_train = r2_score(y_train_orig, y_train_pred_orig)
    r2_test = r2_score(y_test_orig, y_test_pred_orig)
```

Resultado mejor modelo sin kfold:

<img width="762" alt="image" src="https://github.com/user-attachments/assets/7bdaf850-3dde-4169-99d1-b980c467bb19" />

Boxplot con diferentes grados de polinomio con validación cruzada:

```python
# Definir rango de grados de polinomio a probar
degrees = range(1, 16)

# Configurar validación cruzada con K-Fold
kf = KFold(n_splits=5, shuffle=True, random_state=42)

# Almacenar resultados para los boxplots de RMSE y R²
rmse_scores = {degree: [] for degree in degrees}
r2_scores = {degree: [] for degree in degrees}

# Iterar sobre cada grado de polinomio
for degree in degrees:
    poly = PolynomialFeatures(degree)
    X_poly = poly.fit_transform(X_interp)  # Aplicar transformación polinómica

    # Modelo Ridge con alpha fijo
    alpha = 0.01  # Se puede ajustar si es necesario
    model = Ridge(alpha=alpha)

    # Validación cruzada con RMSE y R²
    mse_cv_scores = cross_val_score(model, X_poly, y_interp, cv=kf, scoring=make_scorer(mean_squared_error))
    r2_cv_scores = cross_val_score(model, X_poly, y_interp, cv=kf, scoring=make_scorer(r2_score))

    # Guardar RMSE (Raíz de MSE) y R²
    rmse_scores[degree] = np.sqrt(np.abs(mse_cv_scores))  # Convertir MSE a RMSE
    r2_scores[degree] = np.abs(r2_cv_scores)  # Convertir R² a valores absolutos para escala logarítmica
```

<img width="1019" alt="image" src="https://github.com/user-attachments/assets/a7fbd12c-a27e-483d-a0b9-ed18267f000e" />

Mejor modelo con kfold:

<img width="761" alt="image" src="https://github.com/user-attachments/assets/f8cde2bf-34b8-4360-8472-37d49899649c" />

Mejor modelo con desviación estandar extendido para predicciones:

<img width="770" alt="image" src="https://github.com/user-attachments/assets/24e7d2de-8d4b-4265-9eb0-06174d12974f" />

Importancia de los pesos en el modelo: 

<img width="742" alt="image" src="https://github.com/user-attachments/assets/947bfb0a-7a6f-41bd-a928-ed261cced4f7" />


<img width="746" alt="image" src="https://github.com/user-attachments/assets/a92c89c5-9475-499b-992c-f08c8d8f343a" />

Importancia de pesos en diferentes modelos:

<img width="1038" alt="image" src="https://github.com/user-attachments/assets/844c7bc2-8c5f-4383-ad4a-668fa864cf26" />

<img width="1021" alt="image" src="https://github.com/user-attachments/assets/660a1063-2d83-4fc2-997f-e52613c19670" />

<img width="1028" alt="image" src="https://github.com/user-attachments/assets/ae75f0ea-7cd5-4cfd-9b19-13b709f301f0" />


## Modelo de clasificación por riesgo de abandonar la plataforma

```python
# Crear las columnas necesarias en ambos datasets
for df, user_col in zip([usuarios_activos, usuarios_eliminados], ["user_id", "deleted_account_id"]):
    # Contar total de compras por usuario
    df["total_compras"] = df.groupby(user_col)["id"].transform("count")

    # Sumar el monto total gastado por usuario
    df["monto_total"] = df.groupby(user_col)["amount"].transform("sum")

    # Eliminar duplicados para que cada usuario tenga solo una fila
    df.drop_duplicates(subset=[user_col], inplace=True)
```

Cluster de datos:

```python
# Seleccionar características para la clusterización
X = df[["total_compras", "monto_total"]].copy()

# Escalar los datos para mejor normalización
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Aplicar K-Means con 3 clusters
kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
df["cluster_kmeans"] = kmeans.fit_predict(X_scaled)

# Aplicar DBSCAN
dbscan = DBSCAN(eps=0.8, min_samples=5)  # Ajustar parámetros según datos
df["cluster_dbscan"] = dbscan.fit_predict(X_scaled)
```
<img width="736" alt="image" src="https://github.com/user-attachments/assets/149b2ccf-c9d7-4c51-a8b3-4e4caf0dc5cd" />

<img width="729" alt="image" src="https://github.com/user-attachments/assets/6db7f93a-2639-4386-a833-6c5ead92d7bb" />

<img width="747" alt="image" src="https://github.com/user-attachments/assets/91ed3c04-2f27-4ac2-913c-ed17e8ee2c60" />

<img width="745" alt="image" src="https://github.com/user-attachments/assets/e20e50ec-4055-4437-9173-c7abd3c0ed77" />


<img width="619" alt="image" src="https://github.com/user-attachments/assets/408816c0-94fb-495f-8dc0-f350acd573c0" />


Primer modelo:

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report
from imblearn.over_sampling import SMOTE

# 📌 1️⃣ Cargar los datasets
usuarios_activos = pd.read_csv("drive/MyDrive/ColabNotebooks/Business_Payments/usuarios_activos_procesado.csv")
usuarios_eliminados = pd.read_csv("drive/MyDrive/ColabNotebooks/Business_Payments/usuarios_eliminados_procesado.csv")

# 📌 2️⃣ Convertir variables numéricas que pueden estar en formato string
cols_numericas = ["total_compras", "monto_total", "tasa_rechazo", "intervalo_promedio_solicitudes", "pago_tardio_ratio"]

for df in [usuarios_activos, usuarios_eliminados]:
    for col in cols_numericas:
        df[col] = pd.to_numeric(df[col], errors="coerce")

# 📌 3️⃣ Crear la variable objetivo 'status' (1 = eliminado, 0 = activo)
usuarios_activos["status"] = 0
usuarios_eliminados["status"] = 1

# 📌 4️⃣ Unificar ambos datasets en un solo DataFrame
df_usuarios = pd.concat([usuarios_activos, usuarios_eliminados], ignore_index=True)

# 📌 5️⃣ Seleccionar las características relevantes
features = ["total_compras", "monto_total", "tasa_rechazo", "intervalo_promedio_solicitudes", "pago_tardio_ratio"]
X = df_usuarios[features].fillna(0)  # Rellenar posibles valores nulos con 0
y = df_usuarios["status"]

# 📌 6️⃣ Normalizar las variables numéricas
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 📌 7️⃣ Aplicar SMOTE para balancear las clases
smote = SMOTE(sampling_strategy=0.5, random_state=42)  # Ajustamos para que la clase 1 tenga el 50% de la clase 0
X_resampled, y_resampled = smote.fit_resample(X_scaled, y)

# 📌 8️⃣ Separar datos en entrenamiento y prueba (80% - 20%)
X_train, X_test, y_train, y_test = train_test_split(X_resampled, y_resampled, test_size=0.2, random_state=42, stratify=y_resampled)

# 📌 9️⃣ Entrenar el modelo de Regresión Logística con los datos balanceados
log_reg_smote = LogisticRegression()
log_reg_smote.fit(X_train, y_train)

# 📌 🔟 Realizar predicciones en el conjunto de prueba
y_pred_smote = log_reg_smote.predict(X_test)
y_pred_prob_smote = log_reg_smote.predict_proba(X_test)[:, 1]

# 📌 Evaluar el modelo después de aplicar SMOTE
accuracy_smote = accuracy_score(y_test, y_pred_smote)
conf_matrix_smote = confusion_matrix(y_test, y_pred_smote)
class_report_smote = classification_report(y_test, y_pred_smote)
```
Resultado:

<img width="420" alt="image" src="https://github.com/user-attachments/assets/ce1f1fed-3a4c-458f-953a-20e1c56b6b03" />

## Modelo tras agregar variables de kmeans y kernel RBF

<img width="538" alt="image" src="https://github.com/user-attachments/assets/97caad13-7b2d-44df-a091-0fb012b9bcb9" />

Importancia variables:

<img width="905" alt="image" src="https://github.com/user-attachments/assets/5e8eff70-948f-4ce2-a15a-49ed1f0cce12" />

## Aplicar el modelo

```python
# 📌 1️⃣ Cargar los datos de usuarios activos actuales
usuarios_activos = pd.read_csv("drive/MyDrive/ColabNotebooks/Business_Payments/usuarios_activos_procesado.csv")

# 📌 2️⃣ Seleccionar las mismas variables predictivas usadas en el modelo
features_extra = [
    "total_compras", "monto_total", "tasa_rechazo",
    "intervalo_promedio_solicitudes", "pago_tardio_ratio", "solicitudes_modificadas", "cluster_kmeans"
]

# Convertir a valores numéricos
for col in features_extra:
    usuarios_activos[col] = pd.to_numeric(usuarios_activos[col], errors="coerce")

# 📌 3️⃣ Normalizar las variables usando el mismo escalador entrenado
X_activos = usuarios_activos[features_extra].fillna(0)
X_scaled_activos = scaler_extra.transform(X_activos)  # Usa el scaler ya entrenado

# 📌 4️⃣ Aplicar el modelo para predecir la probabilidad de eliminación
usuarios_activos["probabilidad_eliminacion"] = best_model.predict_proba(X_scaled_activos)[:, 1]

# 📌 5️⃣ Clasificar usuarios en niveles de riesgo
def clasificar_riesgo(prob):
    if prob > 0.7:
        return "Alto Riesgo 🔴"
    elif prob > 0.4:
        return "Riesgo Moderado 🟡"
    else:
        return "Bajo Riesgo 🟢"

usuarios_activos["nivel_riesgo"] = usuarios_activos["probabilidad_eliminacion"].apply(clasificar_riesgo)
```

<img width="527" alt="image" src="https://github.com/user-attachments/assets/82e4bc52-f008-4a85-b514-a79c750f2131" />


## Modelo con XGBOOST

Dependencias a instalar en colab:

```bash
!pip uninstall -y xgboost scikit-learn imbalanced-learn mlxtend
!pip install --no-cache-dir scikit-learn==1.3.2 imbalanced-learn==0.13.0 mlxtend xgboost==1.7.4
```

```python
# Importar librerías necesarias
import pandas as pd
import numpy as np
from xgboost import XGBClassifier
from sklearn.model_selection import train_test_split, GridSearchCV, StratifiedKFold
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report
from imblearn.over_sampling import SMOTE

# 📌 1️⃣ Cargar los datasets procesados
usuarios_activos = pd.read_csv("drive/MyDrive/ColabNotebooks/Business_Payments/usuarios_activos_procesado.csv")
usuarios_eliminados = pd.read_csv("drive/MyDrive/ColabNotebooks/Business_Payments/usuarios_eliminados_procesado.csv")

# 📌 2️⃣ Crear la variable objetivo 'status' (1 = eliminado, 0 = activo)
usuarios_activos["status"] = 0
usuarios_eliminados["status"] = 1

# Unificar ambos datasets en un solo DataFrame
df_usuarios = pd.concat([usuarios_activos, usuarios_eliminados], ignore_index=True)

# 📌 3️⃣ Selección de Variables Predictivas
features_xgb = [
    "total_compras", "monto_total", "tasa_rechazo",
    "intervalo_promedio_solicitudes", "pago_tardio_ratio", "solicitudes_modificadas","cluster_kmeans"
]

# Convertir a valores numéricos (por si hay strings)
for col in features_xgb:
    df_usuarios[col] = pd.to_numeric(df_usuarios[col], errors="coerce")

# Seleccionar variables y objetivo
X_xgb = df_usuarios[features_xgb].fillna(0)  # Rellenar valores NaN con 0
y_xgb = df_usuarios["status"]

# 📌 4️⃣ Normalizar las variables numéricas
scaler_xgb = StandardScaler()
X_scaled_xgb = scaler_xgb.fit_transform(X_xgb)

# 📌 5️⃣ Aplicar SMOTE para balancear clases
smote = SMOTE(sampling_strategy=0.5, random_state=42)  # Ajustamos a 50% de la clase mayoritaria
X_resampled_xgb, y_resampled_xgb = smote.fit_resample(X_scaled_xgb, y_xgb)

# 📌 6️⃣ Separar en entrenamiento y prueba (80% - 20%)
X_train_xgb, X_test_xgb, y_train_xgb, y_test_xgb = train_test_split(
    X_resampled_xgb, y_resampled_xgb, test_size=0.2, random_state=42, stratify=y_resampled_xgb
)

# 📌 7️⃣ Configurar hiperparámetros iniciales para XGBoost
xgb_model = XGBClassifier(
    objective='binary:logistic',
    eval_metric='logloss',
    use_label_encoder=False,
    random_state=42
)

# 📌 8️⃣ Definir los Hiperparámetros a Ajustar
param_grid_xgb = {
    "n_estimators": [50, 100, 200],  # Número de árboles en el modelo
    "max_depth": [3, 5, 7],  # Profundidad máxima de cada árbol
    "learning_rate": [0.01, 0.1, 0.2],  # Tasa de aprendizaje
    "subsample": [0.8, 1.0],  # Proporción de datos usados en cada iteración
    "colsample_bytree": [0.8, 1.0]  # Proporción de columnas usadas en cada iteración
}

# 📌 9️⃣ Entrenar XGBoost con GridSearchCV
grid_search_xgb = GridSearchCV(
    xgb_model, param_grid_xgb, cv=StratifiedKFold(n_splits=5), scoring="accuracy", n_jobs=-1
)
grid_search_xgb.fit(X_train_xgb, y_train_xgb)

# 📌 🔟 Evaluar el Mejor Modelo Encontrado
best_xgb_model = grid_search_xgb.best_estimator_

# Predicciones con el mejor modelo
y_pred_xgb = best_xgb_model.predict(X_test_xgb)
y_pred_prob_xgb = best_xgb_model.predict_proba(X_test_xgb)[:, 1]

# 📌 1️⃣1️⃣ Ajuste del Umbral de Decisión
nuevo_umbral = 0.5  # Reducimos el umbral para detectar más eliminados
y_pred_umbral = (y_pred_prob_xgb >= nuevo_umbral).astype(int)

# 📌 1️⃣2️⃣ Evaluación del Modelo con Nuevo Umbral
accuracy_xgb = accuracy_score(y_test_xgb, y_pred_umbral)
conf_matrix_xgb = confusion_matrix(y_test_xgb, y_pred_umbral)
class_report_xgb = classification_report(y_test_xgb, y_pred_umbral)

# 📌 1️⃣3️⃣ Mostrar Resultados
print(f"🔹 Mejores Hiperparámetros Encontrados: {grid_search_xgb.best_params_}")
print(f"🔹 Precisión del Modelo XGBoost con Umbral Ajustado: {accuracy_xgb:.4f}")
print("\n🔹 Matriz de Confusión con Nuevo Umbral:")
print(conf_matrix_xgb)
print("\n🔹 Reporte de Clasificación con Nuevo Umbral:")
print(class_report_xgb)
```

<img width="1044" alt="image" src="https://github.com/user-attachments/assets/672b315d-cb89-4a91-b9fb-25fe6878bb33" />

```python
import numpy as np
from sklearn.metrics import confusion_matrix, classification_report

# 📌 1️⃣ Establecer el nuevo umbral
nuevo_umbral = 0.4  # Modificamos el umbral para detectar más eliminados

# 📌 2️⃣ Predecir probabilidades en el conjunto de prueba
y_pred_prob_xgb = best_xgb_model.predict_proba(X_test_xgb)[:, 1]

# 📌 3️⃣ Convertir a etiquetas según el nuevo umbral
y_pred_umbral = (y_pred_prob_xgb >= nuevo_umbral).astype(int)

# 📌 4️⃣ Evaluación del modelo con nuevo umbral
conf_matrix_umbral = confusion_matrix(y_test_xgb, y_pred_umbral)
class_report_umbral = classification_report(y_test_xgb, y_pred_umbral)
```

<img width="430" alt="image" src="https://github.com/user-attachments/assets/c761c8b0-02f5-4ca7-9e71-4dab6d69a02c" />

Métricas de rendimiento del modelo:

<img width="916" alt="image" src="https://github.com/user-attachments/assets/b2636886-b2af-4f44-992e-25952461844c" />

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/3bba7933-df8b-40fb-b2ac-e22deece0a0e" />

<img width="1107" alt="image" src="https://github.com/user-attachments/assets/7ac3f2b4-a56a-4d4b-b905-64a577de7584" />

<img width="1087" alt="image" src="https://github.com/user-attachments/assets/879c02ce-12e4-468a-bdc3-656f9c7d313d" />

<img width="747" alt="image" src="https://github.com/user-attachments/assets/b11633d3-1338-453d-8a4a-e0f4e9dc013a" />

