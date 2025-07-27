# 💰 Estimación de Gastos de Infraestructura Tecnológica

# 📊 Resumen de Costes
```
┌──────────────────────────────────────────────────────────────┐
│                    RESUMEN DE COSTOS MENSUALES               │
├──────────────────────────────────────────────────────────────┤
│ Entorno.      │ Mínimo      │ Recomendado │ Enterprise       │
├───────────────┼─────────────┼─────────────┼──────────────────┤
│ Desarrollo    │ €150        │ €300        │ €500             │
│ Staging       │ €200        │ €400        │ €700             │
│ Producción    │ €500        │ €1,500      │ €3,000           │
├───────────────┼─────────────┼─────────────┼──────────────────┤
│ TOTAL/MES     │ €850        │ €2,200      │ €4,200           │
│ TOTAL/AÑO     │ €10,200     │ €26,400     │ €50,400          │
└──────────────────────────────────────────────────────────────┘
``
# 🖥️ Infraestructura Cloud (AWS/GCP/Azure)

## Servidores de Aplicación

Opción básica (10-50 clientes)

```
Desarrollo:
- t3.medium (2 vCPU, 4GB RAM): €35/mes
- Storage EBS 100GB: €10/mes

Staging:
- t3.large (2 vCPU, 8GB RAM): €70/mes
- Storage EBS 200GB: €20/mes

Producción:
- t3.xlarge (4 vCPU, 16GB RAM) x2: €280/mes
- Application Load Balancer: €25/mes
- Storage EBS 500GB: €50/mes
```