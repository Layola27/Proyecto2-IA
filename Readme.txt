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
- **Evaluación de incidentes financieros** y su impacto en la rentabilidad del servicio.
- **Análisis de cohortes** para entender la retención y el comportamiento de los usuarios a lo largo del tiempo.
- **Modelado predictivo** para detectar posibles anomalías y fraudes financieros.

## Tecnologías y Herramientas Utilizadas
- **Python**: Análisis de datos con `pandas`, `numpy`, `matplotlib`, `seaborn`.
- **Google Colab**: Desarrollo y ejecución de código en la nube.
- **Jupyter Notebook**: Exploración de datos interactiva.
- **Excel**: Revisión y validación de datos adicionales.

## Contribución
Si deseas contribuir a este análisis, puedes proponer mejoras en los enfoques analíticos, visualizaciones o modelos predictivos. Se recomienda seguir buenas prácticas en limpieza y validación de datos antes de realizar cualquier interpretación.

---

Este README sirve como guía introductoria al proyecto y ayudará a comprender la estructura y objetivos del análisis de Business Payments.


