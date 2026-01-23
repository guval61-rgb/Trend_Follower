================================================================================
LECCIONES CRÍTICAS - PROYECTO FOLLOWERPURO
================================================================================
Fecha: 2026-01-21
Proyecto: EA FollowerPuro - Análisis de salidas óptimas

Este documento registra hallazgos críticos y metodología correcta para evitar
repetir errores de análisis.

================================================================================
METODOLOGÍA CORRECTA
================================================================================

REGLA #1: NUNCA ESPECULAR SIN DATOS OBJETIVOS
- ❌ MAL: "Los cruces se activan tarde por asimetría natural del mercado"
- ✅ BIEN: Analizar timing real (bars), pips de activación, comparar BUY vs SELL

REGLA #2: LOS DATOS PUEDEN CONTRADECIR LA INTUICIÓN
- Lo obvio no siempre es correcto
- El mercado no funciona como esperamos
- Medir antes de concluir

REGLA #3: PROCESO DE ANÁLISIS RIGUROSO
1. Plantear hipótesis clara y testable
2. Identificar qué datos necesito
3. Extraer datos del CSV Resumen
4. Calcular promedios/medianas
5. Comparar BUY vs SELL
6. Concluir basado SOLO en datos

================================================================================
HALLAZGO CRÍTICO #1: CRUCES NUEVOS EN SELL
================================================================================

FECHA: 2026-01-21
PROBLEMA: Cruces nuevos (f-l) y combos generan pérdidas masivas en SELL

HIPÓTESIS INICIAL (INCORRECTA):
"Los cruces se activan TARDE (después de P40) y capturan solo el retroceso"

DATOS REALES:
- Cruces nuevos se activan a 12-40 bars (MUY TEMPRANO)
- P40 se activa a 218 bars en SELL
- Los cruces se activan 180-200 bars ANTES de P40
- Se activan con pips NEGATIVOS: -0.5 a -2.3 pips
- TODOS (7/7) los cruces nuevos se activan en pérdida en SELL

CAUSA RAÍZ CONFIRMADA:
Los cruces basados en SMA5 y LWMA20 son HIPERSENSIBLES al ruido natural
de los primeros 12-40 bars. En movimientos SELL (bajistas enérgicos), los
micro-retrocesos naturales activan prematuramente los cruces, cerrando
trades que hubieran alcanzado +40 pips.

EVIDENCIA CUANTITATIVA:
┌─────────────┬──────────┬───────────┬──────────┬───────────┐
│ Cruce       │ BUY Bars │ SELL Bars │ BUY Pips │ SELL Pips │
├─────────────┼──────────┼───────────┼──────────┼───────────┤
│ Cruce_f     │   12.1   │   12.5    │  -0.16   │   -0.57   │
│ Cruce_g     │   24.4   │   22.4    │  -0.24   │   -1.03   │
│ Cruce_h     │   29.8   │   26.2    │  -0.38   │   -0.92   │
│ Cruce_i     │   40.2   │   34.8    │  +0.32   │   -1.47   │
│ Cruce_j     │   29.7   │   26.2    │  +0.24   │   -0.97   │
│ Cruce_k     │   27.4   │   23.9    │  -0.86   │   -0.89   │
│ Cruce_l     │   46.9   │   38.6    │  +0.52   │   -2.31   │
├─────────────┼──────────┼───────────┼──────────┼───────────┤
│ P40 (ref)   │  208.8   │  218.8    │  +43.45  │  +43.61   │
└─────────────┴──────────┴───────────┴──────────┴───────────┘

COMPARACIÓN DE RENDIMIENTO TOTAL:
- SMA5×LWMA20: BUY -52 pips | SELL -193 pips (pérdida 365% mayor en SELL)
- SMA5×LWMA50: BUY -81 pips | SELL -347 pips (pérdida 328% mayor en SELL)
- P40:         BUY +6,604 pips | SELL +6,541 pips (similar rendimiento)

CONCLUSIÓN:
El problema NO es un bug de código. Es conceptual: las MAs rápidas son
incompatibles con la volatilidad temprana de movimientos SELL.

DECISIÓN PARA EA FOLLOWERPURO:
Para SELL usar SOLO:
✓ Pips fijos: P30, P40, P50
✓ Cruces lentos: Cruce_a (LWMA200×220 - se activa a 65 bars)
✓ Retrocesos: R20-R50
✗ NO usar cruces nuevos (f-l)
✗ NO usar combos

================================================================================
HALLAZGO CRÍTICO #2: CRUCES ORIGINALES TAMBIÉN AFECTADOS
================================================================================

DESCUBRIMIENTO: Los cruces ORIGINALES (a-e) también son más negativos en
SELL que en BUY, aunque en menor magnitud.

DATOS:
┌─────────────────┬──────────┬───────────┬──────────┬───────────┐
│ Cruce Original  │ BUY Bars │ SELL Bars │ BUY Pips │ SELL Pips │
├─────────────────┼──────────┼───────────┼──────────┼───────────┤
│ Precio×SMA5 (e) │    5.0   │    4.8    │  +0.05   │   -0.33   │
│ LWMA20×22 (d)   │   18.3   │   17.5    │  -0.59   │   -0.50   │
│ LWMA50×53 (c)   │   36.6   │   34.1    │  -0.51   │   -1.43   │
│ LWMA100×105 (b) │   38.9   │   31.8    │  -0.27   │   -2.43   │
│ LWMA200×220 (a) │   69.0   │   64.8    │  -1.38   │   -2.62   │
└─────────────────┴──────────┴───────────┴──────────┴───────────┘

PATRÓN: Cuanto más RÁPIDA la MA, peor rendimiento en SELL.
- MAs rápidas (e,d,c) = activación temprana = pérdidas
- MAs lentas (a) = activación tardía = menor pérdida

IMPLICACIÓN: Para SELL, solo cruces con MAs >100 períodos son viables.

================================================================================
METODOLOGÍA PARA FUTUROS ANÁLISIS
================================================================================

CUANDO ENCUENTRES RESULTADOS NEGATIVOS INESPERADOS:

PASO 1: NO ESPECULAR
- No asumir "timing tardío" sin datos
- No invocar "asimetría natural" sin evidencia
- No crear explicaciones sin medir

PASO 2: IDENTIFICAR DATOS NECESARIOS
Para analizar timing de salidas necesitas:
- XXXXX_Bars: Cuándo se activa (timing absoluto)
- XXXXX_Pips: A qué nivel de pips se activa
- Comparación con referencia conocida (P40, P30)

PASO 3: CREAR SCRIPT DE ANÁLISIS
Script debe calcular:
- Promedio de bars por salida (BUY vs SELL)
- Promedio de pips cuando se activa (BUY vs SELL)
- Diferencia de timing vs salida de referencia
- Conteo de activaciones en ganancia vs pérdida

PASO 4: INTERPRETAR DATOS OBJETIVAMENTE
- Si Bars < P40_Bars → Salida temprana
- Si Bars > P40_Bars → Salida tardía
- Si Pips < 0 cuando activa → Cierra en pérdida
- Si Pips > 0 cuando activa → Cierra en ganancia

PASO 5: VALIDAR HIPÓTESIS
Confirmar que los datos explican el fenómeno observado:
- ¿Los pips negativos en activación explican las pérdidas totales? ✓
- ¿El timing temprano explica por qué se pierden los +40 pips? ✓
- ¿El patrón es consistente en todos los cruces nuevos? ✓

================================================================================
ARCHIVOS CLAVE PARA ANÁLISIS
================================================================================

CSV RESUMEN (MAs_Resumen_EURUSD_M5_2023_06_15.csv):
- Contiene timing detallado (Bars) y pips de cada salida
- Encoding: latin-1
- Columnas críticas: Tipo, Combinacion, XXXXX_Bars, XXXXX_Pips

CSV SEÑALES (MAs_Señales_EURUSD_M5_2023_06_15.csv):
- Registro de cada señal detectada
- Útil para análisis de frecuencia

RANKING (Ranking_Combinacion_X.csv):
- Datos agregados por salida
- Útil para comparaciones rápidas
- NO contiene timing detallado

SCRIPT DE DIAGNÓSTICO:
- diagnostico_timing_cruces.py
- Analiza timing y pips de activación
- Compara BUY vs SELL
- Genera Diagnostico_Timing_Salidas.csv

================================================================================
CONFIGURACIÓN ESTÁNDAR
================================================================================

ENCODING:
- CSVs Windows: latin-1
- Siempre incluir fallback UTF-8 → latin-1 → iso-8859-1

SEPARADORES:
- CSV estándar: coma (,)
- TSV/TXT: tabulador (\t)

TIMEFRAME:
- M5 para análisis detallado
- H4 para análisis de largo plazo

COMBINACIONES:
- A-E: Originales (5)
- F-M: Agregadas v1.06 (8)
- Total: 13 combinaciones

SALIDAS RASTREADAS:
- Pips fijos: 19
- Retrocesos: 7
- Cruces originales: 5
- Cruces nuevos: 7
- Combos: 6
- Timeouts: 17
- Stop Loss: 20
- Señal contraria: 1
- TOTAL: 82 salidas virtuales simultáneas

================================================================================
LECCIONES PARA CLAUDE (IA)
================================================================================

1. NUNCA confiar en intuición inicial sin validar con datos
2. Patrones "obvios" pueden ser completamente incorrectos
3. La dirección del movimiento (BUY/SELL) afecta profundamente el timing
4. MAs rápidas ≠ salidas rápidas | pueden ser hipersensibles al ruido
5. SIEMPRE solicitar archivo CSV Resumen para análisis de timing
6. SIEMPRE comparar BUY vs SELL por separado
7. SIEMPRE calcular promedios de Bars Y Pips de activación
8. Especular sin datos = pérdida de tiempo y credibilidad

================================================================================
CONTACTO Y ACTUALIZACIONES
================================================================================

Este documento debe actualizarse cuando:
- Se descubran nuevos patrones críticos
- Se identifiquen errores de análisis
- Se validen nuevas hipótesis con datos

Último hallazgo: 2026-01-21
Próxima revisión: Después de análisis de otras combinaciones (L, M, J)

================================================================================
FIN DEL DOCUMENTO
================================================================================
