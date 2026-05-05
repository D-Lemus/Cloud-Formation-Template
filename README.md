# Infraestructura AWS

Proyecto de arquitectura en la nube para el simulador de exámenes de universidad.
Materia: Arquitectura en la Nube · ITESO · Primavera 2026

**Integrantes**
- Diego Lemus
- Josué Godoy
- Sebastián Sánchez Arana

---

## Descripción del Proyecto

Esta Infraestructura es para una aplicación web de preparación para el examen de admisión universitaria, dirigida a estudiantes de preparatoria en México. La vision que tenemos a futuro es que la plataforma ofrezca hasta 30 preguntas diarias por tema con retroalimentación impulsada por inteligencia artificial.

---

## Arquitectura

```
Internet
    │
    ▼
Application Load Balancer (Public Subnet us-east-1a / us-east-1b)
    │
    ├──▶ EC2 Node.js #1 (Private Subnet us-east-1a)
    │
    └──▶ EC2 Node.js #2 (Private Subnet us-east-1b)
                │
                ▼
         VPC Endpoint (Gateway)
                │
                ▼
           DynamoDB

Bastion Host (Public Subnet us-east-1a)
    └── SSH access to private EC2s
```

---

## Recursos que crea el Template

| Recurso | Tipo AWS | Descripción |
|---|---|---|
| VPC | `AWS::EC2::VPC` | Red privada `10.0.0.0/16` |
| Internet Gateway | `AWS::EC2::InternetGateway` | Acceso a internet |
| PublicSubnet | `AWS::EC2::Subnet` | `10.0.1.0/24` · us-east-1a |
| PublicSubnet2 | `AWS::EC2::Subnet` | `10.0.2.0/24` · us-east-1b |
| PrivateSubnet1 | `AWS::EC2::Subnet` | `10.0.3.0/24` · us-east-1a |
| PrivateSubnet2 | `AWS::EC2::Subnet` | `10.0.4.0/24` · us-east-1b |
| PublicRouteTable | `AWS::EC2::RouteTable` | Ruta al Internet Gateway |
| PrivateRouteTable | `AWS::EC2::RouteTable` | Solo tráfico interno |
| SG Load Balancer | `AWS::EC2::SecurityGroup` | Puerto 80 y 443 desde internet |
| SG Bastion Host | `AWS::EC2::SecurityGroup` | Puerto 22 desde internet |
| SG Private Instances | `AWS::EC2::SecurityGroup` | Puerto 3000 desde LB, 22 desde Bastion |
| Bastion Host | `AWS::EC2::Instance` | t3.micro · acceso SSH a red privada |
| PrivateInstance1 | `AWS::EC2::Instance` | t3.medium · Node.js en us-east-1a |
| PrivateInstance2 | `AWS::EC2::Instance` | t3.medium · Node.js en us-east-1b |
| VPC Endpoint DynamoDB | `AWS::EC2::VPCEndpoint` | Conexión privada a DynamoDB sin salir a internet |
| Target Group | `AWS::ElasticLoadBalancingV2::TargetGroup` | Apunta a las dos EC2 en puerto 3000 |
| Load Balancer | `AWS::ElasticLoadBalancingV2::LoadBalancer` | ALB internet-facing |
| Listener | `AWS::ElasticLoadBalancingV2::Listener` | Puerto 80 → Target Group |

---

## Prerequisitos

Antes de desplegar el template necesitas tener:

1. Una cuenta de AWS con permisos para crear los recursos listados arriba
2. Un Key Pair `.pem` creado en la región `us-east-1`
3. AWS CLI instalado (opcional, para deployments futuros)

---

## Cómo desplegar

### Opción 1 — Consola de AWS

1. Ve a **AWS Console → CloudFormation → Create Stack**
2. Selecciona **Upload a template file**
3. Sube el archivo `infraestructura.yaml`
4. Dale un nombre al stack (ej. `final-proyect`)
5. En **Stack failure options** deja seleccionado **Roll back all stack resources**
6. Click en **Submit**
7. Espera ~3-4 minutos a que aparezca **CREATE_COMPLETE**

### Opción 2 — AWS CLI

```bash
aws cloudformation deploy \
  --template-file infraestructura.yaml \
  --stack-name final-proyect \
  --region us-east-1
```

---

## Seguridad

- Las EC2 de aplicación están en **subredes privadas** — no son accesibles directamente desde internet
- El acceso SSH a las instancias privadas se hace únicamente a través del **Bastion Host**
- DynamoDB se accede exclusivamente a través del **VPC Endpoint Gateway** — el tráfico nunca sale a internet
- Los Security Groups siguen el **principio de mínimo privilegio** — cada recurso solo acepta el tráfico estrictamente necesario
- Las credenciales de AWS se manejan mediante **IAM Roles** asignados a las instancias — sin credenciales escritas en el código

---

## Consideraciones de Costo

> ⚠️ Los siguientes recursos generan costo mientras estén activos:
> - 3 instancias EC2 (t3.micro × 1, t3.medium × 2)
> - Application Load Balancer
>
> Recuerda hacer **Delete Stack** en CloudFormation cuando no estés usando la infraestructura para evitar costos innecesarios.

---

## Estructura del Repositorio

```
├── infraestructura.yaml   # Template de CloudFormation
└── README.md              # Este archivo
```
