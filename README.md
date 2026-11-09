# Agente de Conciliación de Cobros para una promotora inmobiliaria

Checkpoint 1 del curso AI Automation Avanzado (Coderhouse) · Natalia Lusso

Primera versión de un sistema de cobranza para una promotora residencial en España. La empresa, los clientes y los importes son ficticios.

## Problema

Cada mes, Administración descarga el extracto del banco, busca a qué cliente corresponde cada ingreso, comprueba si pagó lo que debía, actualiza su estado de cuenta y lo prepara con el formato de la empresa. Es un trabajo manual, lento y propenso a errores.

## Qué hace el agente

Alguien de Administración escribe, por ejemplo: `Necesito el estado de cuenta actualizado de Laura Demo Pérez`. El agente:

1. Identifica el contrato del cliente.
2. Busca en el extracto los ingresos de ese cliente que aún no están conciliados.
3. Registra los que coinciden sin duda y envía a revisión humana los dudosos (pago de más, varios contratos posibles o concepto que indica otra cuota).
4. Prepara la pestaña **Estado de cuenta** con el formato de la empresa.
5. Responde con la situación actualizada y deja un registro de auditoría en Slack.

## Flujo

```text
Chat Trigger → AI Agent (Tools Agent, máx. 10 iteraciones) ─┬→ Preparar log → Slack #cobranza-log
                 │        │                                 └→ Respuesta al chat
          OpenAI GPT-4o  Memoria de sesión
                 Tools (Google Sheets): consultar_contratos · consultar_movimientos_pendientes
                                        registrar_movimiento · preparar_estado_cuenta
```

## Qué decide la IA y qué no

| Parte | Tipo | Dónde |
|---|---|---|
| Elegir herramientas, identificar al cliente y clasificar cada ingreso | Probabilística | AI Agent |
| Imputar pagos (intereses primero, cuota más antigua después), saldos y vencidos | Determinista | Fórmulas de la hoja |
| Registro de auditoría | Determinista | Preparar log + Slack |

## Cumplimiento de la consigna

| Requisito | Implementación |
|---|---|
| Trigger | Chat Trigger |
| Agente | AI Agent en modo Tools Agent con OpenAI GPT-4o |
| Guardrail | Máximo de 10 iteraciones; máximo 3 movimientos por petición |
| System Prompt | Rol, Ámbito, Objetivo, Proceso, Tabla de decisión, Reglas, Límites y Escalamiento |
| Tools | 4 herramientas nativas de Google Sheets conectadas lateralmente, cada una con descripción de cuándo usarla y cuándo no |
| Observabilidad | Mensaje "Tarea completada" en Slack con estado, herramientas usadas, movimientos procesados y respuesta |

## Cómo probarlo

1. Importa `checkpoint1_natalia_lusso.json` en n8n.
2. Haz una copia de la hoja de ejemplo en tu Google Drive y actualiza su ID en las cuatro herramientas.
3. Asigna credenciales de OpenAI, Google Sheets y Slack, y crea el canal `#cobranza-log`.
4. Abre el chat y prueba: `Necesito el estado de cuenta actualizado de Laura Demo Pérez`.

## Pruebas realizadas

| Petición | Resultado | Log en Slack |
|---|---|---|
| `Necesito el estado de cuenta actualizado de Construcciones Demo S.L.` | Registra MOV-0042 (2.904,00 €, importe exacto); el cliente queda al corriente y el estado de cuenta se actualiza | PAGOS REGISTRADOS (1) |
| `Estado de cuenta de Marco Demo Ruiz` | Envía MOV-0045 (7.260,00 €) a revisión: el concepto indica una cuota posterior a la pendiente; el saldo no cambia | REQUIERE REVISIÓN HUMANA (1) |

## Evolución prevista

- **M2:** Manager y Workers (Conciliación y Estado de cuenta), con reglas fijas de coincidencia y aprobación de pagos dudosos desde Slack.
- **M3:** memoria en Airtable por sesión.
- **M4:** correo entrante, CRM y borradores de respuesta al cliente.

Este repositorio no contiene claves ni datos reales.
