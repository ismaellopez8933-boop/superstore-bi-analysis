# 📊 Superstore Sales & Profitability Analytics (Power BI Portfolio Project)

Un proyecto integral de Business Intelligence desarrollado para auditar el desempeño comercial, la rentabilidad neta y la dinámica de clientes de la cadena minorista **Global Superstore**. El proyecto aborda el reto habitual de transformar un conjunto de datos transaccional plano (*flat table*) en un modelo dimensional interactivo (*Star Schema*) con medidas calculadas en DAX y una interfaz ejecutiva estructurada en cuadrícula.

---

## 🎯 Objetivos de Negocio

1. **Monitoreo Financiero y Rentabilidad Operativa:** Contrastar el volumen facturado (`Sales`) frente a la ganancia líquida retenida (`Profit`), controlando el margen porcentual (`% de ganancia`) para detectar inconsistencias de escalado y estacionalidad trimestral.
2. **Dinámica y Concentración de Clientes:** Analizar la distribución de ventas según segmento de mercado (*Consumer*, *Corporate*, *Home Office*) e identificar cuentas clave mediante rankings interactivos y análisis territorial por región comercial.
3. **Optimización del Portafolio de Productos:** Analizar el comportamiento del catálogo por categorías y subcategorías, identificando tanto artículos tractores como productos con bajo desempeño de ventas o inventario de lenta rotación.

---

## 🔄 Transformación de Datos: De Tabla Plana a Modelo Dimensional

El dataset original consistía en un archivo tabular desnormalizado con múltiples columnas repetitivas. A través de **Power Query (M)** se aplicó un proceso de extracción, limpieza y estructuración dimensional:

1. **Corrección de Configuración Regional (*Culture Locale*):** Se corrigió la interpretación de separadores decimales estadounidenses (punto flotante `.`) frente a configuraciones locales para evitar distorsiones de escala entre los campos de ingresos y ganancias.
2. **Desacoplamiento Dimensional:**
   * Se extrajeron las entidades descriptivas `client` y `product` eliminando duplicados sobre sus identificadores únicos (`Customer ID`, `Product ID`).
   * Se depuró la dimensión de productos para conservar exclusivamente atributos categóricos, eliminando métricas transaccionales duplicadas.
   * La tabla `sale` se consolidó como tabla de hechos central que retiene las transacciones, claves foráneas y variables cuantitativas (`Sales`, `Profit`, `Quantity`, `Discount`).
3. **Aislamiento de Lógica Analítica:** Se creó una tabla desacoplada (`Medidas`) para centralizar las fórmulas DAX y mantener un entorno de trabajo limpio.

---

## 🏗️ Arquitectura del Modelo de Datos (Star Schema)

El modelo dimensional implementado responde a un esquema en estrella puro, garantizando filtrado unidireccional y un consumo óptimo de memoria en el motor VertiPaq:

```
          ┌────────────────────────┐
          │         client         │
          │────────────────────────│
          │ PK  Customer ID        │
          │     Customer Name      │
          │     Segment            │
          │     Country / City     │
          │     State / Postal Code│
          │     Region             │
          └───────────┬────────────┘
                      │ 1
                      │
                      │ *
┌─────────────────────┴──────────────────────┐          ┌──────────────────────┐
│                    sale                    │ *      1 │       product        │
│────────────────────────────────────────────│──────────│──────────────────────│
│ PK/FK  Order ID                            │          │ PK  Product ID       │
│ FK     Customer ID                         │          │     Product Name     │
│ FK     Product ID                          │          │     Category         │
│        Order Date                          │          │     Sub-Category     │
│        Sales                               │          └──────────────────────┘
│        Quantity                            │
│        Discount                            │
│        Profit                              │
└─────────────────────┬──────────────────────┘
                      │
                      ▼
          ┌────────────────────────┐
          │        Medidas         │
          │────────────────────────│
          │ [Ganancia]             │
          │ [% de ganancia]        │
          └────────────────────────┘
```

### Relaciones y Cardinalidad
* **`client` (1) ➔ `sale` (\*):** Relación de uno a varios vinculada por `Customer ID`.
* **`product` (1) ➔ `sale` (\*):** Relación de uno a varios vinculada por `Product ID`.
* **`Medidas`:** Contenedor centralizado para cálculos de negocio (sin relaciones físicas activas).

---

## 📐 Lógica de Negocio y Medidas DAX

Para proteger la estabilidad del reporte ante divisiones por cero o contextos de filtro vacíos, las medidas clave se formularon mediante variables locales (`VAR`) y la función defensiva `DIVIDE`:

### 1. Margen Operativo Porcentual
Evalúa el porcentaje real retenido por cada unidad monetaria facturada:

$$\text{Margen de Ganancia} = \frac{\sum(\text{Profit})}{\sum(\text{Sales})}$$

```dax
% de ganancia = 
VAR totalVentas = SUM(sale[Sales])
VAR totalGanancia = SUM(sale[Profit])
RETURN 
    DIVIDE(totalGanancia, totalVentas, 0)
```

### 2. Ganancia Neta Consolidada
```dax
Ganancia = SUM(sale[Profit])
```

---

## 🖥️ Estructura del Tablero Ejecutivo (Power BI)

El diseño del tablero implementa un sistema modular de cuadrícula (2x2) con paleta corporativa coherente, panel de navegación lateral persistente y tarjetas de fondo (*cards*) para delimitar cada componente visual:

### 1. 📈 Página: Ventas (Visión Macro & Temporal)
* **KPIs Directivos:** Tarjetas de alto impacto con *Ganancia Total* formateada y *% de Ganancia*.
* **Evolución Temporal:** Gráficos de líneas paralelas para comparar el comportamiento estacional de ventas y ganancias a lo largo de los años.
* **Filtro Temporal Dinámico:** Selector deslizante (*slider*) por `Order Date`.
* **Matriz de Rendimiento Trimestral:** Tabla cruzada que desglosa ventas e ingresos por año y trimestre (Q1–Q4) con totales marginales.

### 2. 👥 Página: Clientes (Segmentación & Territorio)
* **Top 20 Clientes Interactivo:** Gráfica de barras horizontales controlada mediante marcadores (*Bookmarks*) para alternar en un solo clic entre ordenamiento por **Ventas** o por **Ganancias**.
* **Segmentación de Cartera:** Gráfico de dona con indicador central de ventas consolidadas (`$1,131.59 mill.`), reflejando la proporción entre *Consumer*, *Corporate* y *Home Office*.
* **Desempeño Regional:** Gráfica de barras descendente que compara de forma intuitiva los territorios comerciales (*West*, *Central*, *East*, *South*).
* **Consola de Filtros:** Panel lateral dedicado para filtrado cruzado instantáneo por región y segmento.

### 3. 📦 Página: Productos (Diagnóstico de Catálogo)
* **Pareto de Artículos:**
  * *Top 10 productos más vendidos* (con switch interactivo por ventas o margen).
  * *Top 10 productos con menor venta* (ordenamiento ascendente para identificar referencias de baja rotación).
* **Distribución por Categoría:** Gráfico de dona con indicador central consolidado (`$1,131.59 mill.`) y participación por sector (*Technology*, *Furniture*, *Office Supplies*).
* **Jerarquía por Subcategoría:** Gráfica de barras verticales ordenadas de mayor a menor para una lectura ejecutiva limpia de las familias de producto (*Bookcases*, *Chairs*, *Phones*, etc.).
* **Consola de Filtros:** Segmentación dedicada por categoría en tarjeta unificada.

---

## 🛠️ Stack Tecnológico

* **Power BI Desktop:** Modelado de datos relacional, diseño UX/UI en cuadrícula, marcadores (*Bookmarks*) para vistas alternables y cálculos DAX.
* **Power Query (M):** Normalización de datos, extracción de dimensiones, eliminación de duplicados y ajuste de configuración regional.
* **Excel:** Conciliación preliminar y revisión exploratoria de las columnas de la tabla transaccional.

---

## 📂 Organización del Repositorio

```text
├── data/
│   └── Sample - Superstore.csv    <- Archivo CSV original (tabla plana)
├── pbix/
│   └── SuperStore.pbix  <- Tablero interactivo de Power BI
├── docs/
│   └── img/                       <- Capturas en alta resolución de las 3 vistas
│       ├── Ventas.png                
|       ├── Clientes.png
|       └── Productos.png
└── README.md
```

---

## 🚀 Cómo Explorar este Proyecto

1. Clona el repositorio:
   ```bash
   git clone https://github.com/ismaellopez8933-boop/superstore-bi-analysis.git
   ```
2. Abre el archivo `.pbix` ubicado en la carpeta `pbix/` utilizando **Power BI Desktop**.
3. Utiliza la barra de navegación lateral izquierda para recorrer las pantallas de **Ventas**, **Clientes** y **Productos**, y experimenta con los botones de alternancia en los rankings de clientes y productos.
