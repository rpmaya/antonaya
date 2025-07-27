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

Opción media (50-200 clientes)
```
Desarrollo:
- t3.large: €70/mes
- Storage 200GB: €20/mes

Staging:
- m5.xlarge (4 vCPU, 16GB RAM): €155/mes
- Storage 500GB: €50/mes

Producción:
- m5.2xlarge (8 vCPU, 32GB RAM) x3: €930/mes
- Auto Scaling Groups: incluido
- ALB + WAF: €75/mes
- Storage 1TB: €100/mes
```

Opción enterprise (200+ clientes)
```
Desarrollo:
- m5.xlarge: €155/mes
- Storage 500GB: €50/mes

Staging:
- m5.2xlarge: €310/mes
- Storage 1TB: €100/mes

Producción:
- c5.4xlarge (16 vCPU, 32GB RAM) x4: €2,200/mes
- Multi-AZ deployment: incluido
- ALB + WAF + Shield: €150/mes
- Storage 5TB: €500/mes
```