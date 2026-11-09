Reto Fullstack — Liquidador de Estacionamiento
Nov 8, 2026 · @carlos
Contexto y reglas del reto
Tienes 5 días calendario para construir un sistema fullstack que calcule el cobro de los tickets de Parqueo Quilla, un estacionamiento ficticio. El cobro debe seguir al pie de la letra el Reglamento Tarifario de este documento. Lo que se evalúa es que leas el reglamento completo, lo entiendas y lo apliques sin errores; la estética no puntúa.
• Plazo: 5 días calendario desde que recibes este documento. Entregas el link a un repositorio git.
• Stack: libre. El proyecto debe levantarse con un solo comando documentado en el README (por ejemplo docker compose up).
• Preguntas: puedes enviarlas hasta el final del día 3. Las preguntas bien planteadas suman puntos. Las respuestas se comparten con todos los participantes.
• Uso de IA: permitido. Declara en IA.md qué herramientas usaste y para qué. En la defensa explicarás tu código sin ayuda y harás un cambio en vivo sobre él.
Entregables
1. Repositorio con historial de commits real (no un único commit al final).
2. README.md: cómo levantar el proyecto y cómo correr los tests.
3. DECISIONES.md: cada punto del reglamento que te pareció ambiguo, cómo lo interpretaste y por qué.
4. IA.md: herramientas de IA usadas y para qué.
5. Tests automatizados de la lógica de cobro, con al menos un test por artículo del reglamento.
Qué hay que construir
Un backend con tres funciones (liquidar, importar y cerrar caja) y un frontend para el operador que las use. El backend debe respetar exactamente el contrato de esta sección, porque se evalúa con pruebas automáticas que lo llaman directamente.
Backend
• Liquidar un ticket: calcula cuánto se cobra según el reglamento y lo guarda.
• Importar un CSV legado: carga tickets desde el formato de la sección "Importación del CSV legado" y reporta qué filas se aceptaron y cuáles se rechazaron, con el motivo.
• Cierre de caja: total cobrado en una fecha. Un ticket pertenece al cierre de la fecha de su salida, en hora de Lima.
• Persistencia: los tickets liquidados e importados deben sobrevivir a un reinicio del servidor.
Frontend
1. Liquidación: formulario para ingresar un ticket y ver el total con su desglose. Cada línea del desglose muestra el artículo que la justifica.
2. Importación: subir el CSV y ver las filas aceptadas y rechazadas, con el motivo de cada rechazo.
3. Cierre: elegir una fecha y ver cantidad de tickets, total cobrado y subtotales por tipo de vehículo y por tickets perdidos.
Contrato: liquidar
POST /api/liquidaciones
{
  "ticket": "T-1001",
  "tipo": "AUTO",
  "entrada": "2026-10-12T10:00:00-05:00",
  "salida": "2026-10-12T11:20:00-05:00",
  "ticket_perdido": false
}
Respuesta 200. Todos los montos van en céntimos enteros. El desglose tiene una línea por cada paso del reglamento que cambió el monto, y cada línea cita su artículo.
{
  "ticket": "T-1001",
  "minutos": 80,
  "total_centimos": 900,
  "detalle": [
    {
      "concepto": "Tramo diurno normal 10:00–11:20",
      "articulo": "Art. 4",
      "monto_centimos": 900
    }
  ]
}
Error 422, con el mismo formato en todos los endpoints:
{ "codigo": "E01", "mensaje": "La salida es anterior a la entrada" }
Código
Cuándo
E01
La salida es anterior a la entrada
E02
El tipo de vehículo no es AUTO ni MOTO
E03
Una fecha u hora no tiene un formato aceptado
Contrato: importar y cerrar
• POST /api/importaciones: recibe el archivo como multipart/form-data en el campo archivo. Responde con las filas aceptadas (ticket y total en céntimos) y las rechazadas (número de fila, ticket y motivo).
• GET /api/cierre?fecha=2026-10-12: responde con cantidad, total_centimos y los subtotales de la pantalla de cierre. El diseño exacto de ambas respuestas lo decides tú y lo documentas en el README.
Reglamento Tarifario de Parqueo Quilla (versión 3)
Este es el reglamento que tu sistema debe aplicar. Léelo completo antes de programar: las adendas del final modifican artículos anteriores.
Art. 1. Moneda y hora. Los montos están en soles (S/). Toda hora se interpreta en hora de Lima (UTC−05:00, sin horario de verano).
Art. 2. Permanencia. La permanencia son los minutos enteros entre la entrada y la salida. Antes de calcularla, se descartan los segundos de ambas marcas de hora.
Art. 3. Franjas y tramos. La franja diurna va de 06:00 a 22:00. La franja nocturna va de 22:00 a 06:00 del día siguiente. Cada permanencia se divide en tramos consecutivos, uno por cada franja que atraviesa. Cada tramo pertenece a la fecha calendario en que empieza.
Art. 4. Tarifa diurna normal. Se aplica a los tramos diurnos que empiezan de lunes a sábado, siempre que la fecha no sea feriado. La primera hora o fracción cuesta S/ 6.00. Luego, cada fracción de 15 minutos cuesta S/ 1.50.
Art. 5. Tarifa diurna especial. Se aplica a los tramos diurnos que empiezan un domingo o en un feriado del Anexo A. Tiene la misma estructura que el Art. 4, con S/ 5.00 por la primera hora o fracción y S/ 1.00 por cada fracción de 15 minutos.
Art. 6. Tarifa nocturna. Cada tramo nocturno cuesta S/ 10.00, sin importar su duración.
Art. 7. Tope diario. La suma de los tramos que pertenecen a una misma fecha no puede superar S/ 35.00.
Art. 8. Motocicletas. Las motocicletas pagan el 50% del monto.
Art. 9. Redondeo. El monto se redondea hacia abajo al múltiplo de S/ 0.10 más cercano.
Art. 10. Ticket perdido. Si el cliente perdió su ticket, se suma un cargo fijo de S/ 25.00. Este cargo no tiene descuentos ni redondeo.
Art. 11. Tolerancia. Si la permanencia es de 10 minutos o menos, no se cobran los Arts. 4 a 9. El cargo del Art. 10 sí se cobra.
Art. 12. Orden de aplicación. El cobro se calcula en este orden:
1. Tolerancia (Art. 11).
2. Tramos y sus tarifas (Arts. 3 a 6).
3. Tope diario por fecha (Art. 7).
4. Descuento de motocicletas (Art. 8).
5. Redondeo (Art. 9).
6. Cargo por ticket perdido (Art. 10).
Adenda 1. Primera hora. La "primera hora o fracción" de los Arts. 4 y 5 se cobra una sola vez por permanencia, en el primer tramo diurno. Los demás tramos diurnos de la misma permanencia se cobran desde el minuto 0, por fracción de 15 minutos, a la tarifa que les corresponda.
Adenda 2. Tolerancia, vigente para entradas desde el 01/10/2026 a las 00:00. El Art. 11 pasa a ser de 15 minutos o menos y deja de aplicarse a los tickets perdidos. Las entradas anteriores a esa fecha siguen con el Art. 11 original.
Anexo A. Feriados 2026. Solo cuentan estas fechas, aunque exista otro feriado oficial:
Fecha
Feriado
01/01/2026
Año Nuevo
02/04/2026
Jueves Santo
03/04/2026
Viernes Santo
01/05/2026
Día del Trabajo
29/06/2026
San Pedro y San Pablo
28/07/2026
Fiestas Patrias
29/07/2026
Fiestas Patrias
30/08/2026
Santa Rosa de Lima
08/10/2026
Combate de Angamos
01/11/2026
Todos los Santos
08/12/2026
Inmaculada Concepción
25/12/2026
Navidad
Importación del CSV legado
El sistema anterior exporta un CSV con datos sucios. Tu importador debe aceptar lo válido, rechazar lo inválido con un motivo y liquidar cada fila aceptada con el mismo reglamento que el endpoint de liquidación.
Formato
• Codificación UTF-8, separado por comas, con una fila de cabecera: ticket,placa,tipo,entrada,salida,perdido.
• Fechas con zona (Z o un desfase como -05:00 o +01:00): se convierten a hora de Lima.
• Fechas sin zona (2026-10-12 08:15 o 2026-10-12T08:15:00): se interpretan como hora de Lima.
• Cualquier otro formato de fecha, o una fecha que no existe: la fila se rechaza con E03.
• tipo: AUTO o MOTO, sin distinguir mayúsculas y sin contar espacios alrededor. Cualquier otro valor se rechaza con E02.
• perdido: SI, NO o vacío (vacío equivale a NO), sin distinguir mayúsculas. Cualquier otro valor se rechaza con E04.
• Ticket repetido: vale la última fila del archivo con ese ticket. Las anteriores no se cuentan como rechazadas.
• Una fila con salida anterior a la entrada se rechaza con E01.
Código
Cuándo
E04
El campo perdido no es SI, NO ni vacío (solo en importación)
Archivo de ejemplo (legado_ejemplo.csv)
ticket,placa,tipo,entrada,salida,perdido
T-1001,ABC-123,AUTO,2026-10-12 08:15,2026-10-12 10:40,NO
T-1002,M1-456,moto ,2026-10-12T13:00:00Z,2026-10-12T15:10:00Z,
T-1003,XYZ-789,CAMION,2026-10-12 09:00,2026-10-12 10:00,NO
T-1004,DEF-321,AUTO,12/10/2026 09:00,2026-10-12 10:00,NO
T-1005,GHI-654,AUTO,2026-10-12 11:00,2026-10-12 10:30,NO
T-1001,ABC-123,AUTO,2026-10-12 08:15,2026-10-12 11:40,NO
T-1006,JKL-987,AUTO,2026-10-12 21:50,2026-10-12 22:10,NO
T-1007,M2-111,MOTO,2026-10-12 07:00,2026-10-12 07:14,SI
T-1008,MNO-222,AUTO,2026-10-12 18:00,,NO
Ticket
Resultado esperado
T-1001
Aceptado, S/ 21.00 (vale la fila 7)
T-1002
Aceptado, S/ 6.70
T-1003
Rechazado, E02
T-1004
Rechazado, E03
T-1005
Rechazado, E01
T-1006
Aceptado, S/ 16.00
T-1007
Aceptado, S/ 28.00
T-1008 no está en la tabla a propósito: decide qué hacer con esa fila y justifícalo en DECISIONES.md.
Ejemplos públicos
Tu endpoint de liquidación debe devolver estos totales. Los evaluadores además correrán casos que no están en este documento, así que pasar estos cinco no garantiza el puntaje.
#
Tipo
Entrada (Lima)
Salida (Lima)
Perdido
Total
E1
AUTO
lun 12/10/2026 10:00
lun 12/10/2026 11:20
No
S/ 9.00
E2
AUTO
lun 12/10/2026 10:00
lun 12/10/2026 10:15
No
S/ 0.00
E3
AUTO
dom 18/10/2026 09:00
dom 18/10/2026 11:01
No
S/ 10.00
E4
MOTO
mar 13/10/2026 10:00
mar 13/10/2026 12:00
Sí
S/ 31.00
E5
AUTO
lun 12/10/2026 07:00
mar 13/10/2026 20:00
No
S/ 70.00
Desglose esperado de E5. El orden y la redacción de los conceptos son libres; los montos y los artículos citados no.
Concepto
Artículo
Monto
Tramo diurno normal, 12/10 07:00–22:00
Art. 4
S/ 90.00
Tramo nocturno, 12/10 22:00 – 13/10 06:00
Art. 6
S/ 10.00
Tramo diurno normal, 13/10 06:00–20:00, sin primera hora
Art. 4, Adenda 1
S/ 84.00
Tope diario del 12/10 (S/ 100.00 → S/ 35.00)
Art. 7
−S/ 65.00
Tope diario del 13/10 (S/ 84.00 → S/ 35.00)
Art. 7
−S/ 49.00
Total

S/ 70.00
Cómo se evalúa
Casi dos tercios de la nota dependen de que el cobro sea correcto y de que puedas explicar y cambiar tu propio código en vivo.
Criterio
Peso
Qué se mira
Casos ocultos
35%
Montos correctos en casos que no están en este documento, por API y por CSV
Defensa y cambio en vivo
30%
Que expliques tu código y apliques una adenda nueva sobre él
Decisiones y preguntas
15%
Calidad de DECISIONES.md y de las preguntas enviadas hasta el día 3
Código y tests
10%
Claridad, separación de la lógica de cobro, un test por artículo
Frontend
10%
Que las tres pantallas funcionen y el desglose se entienda
La defensa dura 45 minutos, con tu repositorio abierto y compartiendo pantalla:
1. Recorrido (15 min): te pediremos mostrar en qué parte del código se aplica cada artículo y por qué un caso da el monto que da.
2. Cambio en vivo (20 min): recibirás una adenda nueva del reglamento y la implementarás en tu código, con sus tests. Puedes usar tus herramientas habituales, incluida la IA.
3. Preguntas (10 min): sobre tus decisiones y lo que harías distinto.
Una solución que pasa todos los casos pero que no puedes explicar ni modificar no aprueba.
