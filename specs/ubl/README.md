# specs/ubl: documentos UBL 2.1 de referencia

| Carpeta | Contenido |
|---|---|
| `plantillas/` | XML UBL 2.1 **sin firma** (fuente). No validan contra el XSD porque `ext:ExtensionContent` vacío es inválido (aclaración A-01). |
| `ejemplos/` | Las mismas plantillas **firmadas** con XMLDSig y un certificado de prueba (`python tools/validar_specs.py --firmar`). Validan contra el XSD de OASIS y cuadran al céntimo. |
| `ose-demo/` | Evidencia de la prueba en el **OSE-DEMO de Nubefact**: un XML generado por el servidor y el CDR con el que el OSE lo aceptó (docs/11 § 11.7). |
| `xsd-oasis-ubl-2.1/` | XSD oficiales de OASIS, sin modificar. |

Los XML no llevan comentarios antes del elemento raíz. Algunas herramientas, como `probar-xml.nubefact.com`,
asumen que el primer nodo del documento es `Invoice` o `CreditNote` y fallan si encuentran un comentario
(aclaración A-07). Por eso la descripción de cada ejemplo está aquí.

## Los ejemplos

| Archivo | Escenario | Origen | Importes |
|---|---|---|---|
| `20123456786-01-F001-1.xml` | Factura mayorista al crédito (2 cuotas) con una bonificación (afectación 15), guía de remisión y orden de compra | `specs/ejemplos/json/01-emitir-factura-credito.request.json` | gravadas 2261.00 · IGV 406.98 · gratuitas 30.00 (IGV referencial 5.40) · total 2667.98 |
| `20123456786-01-F001-2.xml` | Factura con ISC, Catálogo N.° 08, sistema 01 (al valor). Escenario SUP-03: ABC importa la bebida y es sujeto del ISC. La tasa de 25 % es ilustrativa; en producción viene de la parametrización vigente (TasaIsc) | `03-emitir-factura-isc.request.json` | línea 1: valor 48.00 · ISC 48.00 × 25 % = 12.00 · base IGV 60.00 · IGV 10.80 · total 171.10 |
| `20123456786-03-B001-1.xml` | Boleta minorista con producto exonerado y bolsas con ICBPER. Total ≤ S/ 700: el adquirente puede no identificarse (VAL-BOL-01, SUP-08). Cálculo desde precio con IGV (VAL-TRB-02) | `02-emitir-boleta-icbper.request.json` | 6 × 4.20 = 25.20 → valor 21.36 + IGV 3.84 · ICBPER 2 × 0.50 = 1.00 (fuera de la base del IGV) · total 33.90 |
| `20123456786-07-FC01-1.xml` | Nota de crédito por devolución de un ítem (Catálogo N.° 09 = 07) que modifica exactamente un comprobante (F001-1). Serie con F porque modifica una factura (VAL-NUM-03) | `04-emitir-nota-credito.request.json` | — |

Las facturas F001-1, F001-2 y la boleta B001-1 las **regenera el servidor** desde las solicitudes JSON en cada build,
con el mismo `DigestValue` (`RegresionXmlReferenciaTest`, RNF-03).

## Datos

El emisor (RUC `20123456786`, ABC SOFTWARE CORP S.A.C.), las direcciones y las razones sociales son **ficticios**,
y la firma usa un **certificado de prueba autofirmado**.

El RUC del adquirente de las facturas y de la nota es `20131312955`, el RUC público de SUNAT. Se usa **solo**
porque los OSE verifican que el RUC del receptor exista en el padrón (error 1083, regla VAL-ID-04); la razón social
y la dirección del cliente siguen siendo ficticias. No representa ninguna operación real.

## Probar un ejemplo en el OSE-DEMO de Nubefact

1. Pegar el XML en <https://probar-xml.nubefact.com/> con el destino **OSE**.
2. Si el OSE responde **1033** ("registrado previamente con otros datos"), el número ya fue usado en el OSE-DEMO,
   que es compartido. Hay que cambiar `cbc:ID` por un número no usado y marcar **Firmar**, porque al editar el
   XML la firma original deja de coincidir.
3. Si el OSE observa la fecha, cambiar `cbc:IssueDate` por la de hoy y marcar **Firmar**.

Lo más directo es generar el comprobante con la vista web del servidor (`http://localhost:8080/`), con una serie
nueva y la fecha de hoy: ese XML se acepta tal cual, con la firma del propio servidor (ver `ose-demo/`).
