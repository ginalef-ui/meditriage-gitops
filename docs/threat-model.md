# Threat Model - Microservicio MediTriage Backend

## Metodología STRIDE

| Categoría | Amenaza | Control de Mitigación |
| --- | --- | --- |
| **Spoofing** | Suplantación de identidad en endpoints de triaje. | Autenticación JWT y TLS mutuo (mTLS). |
| **Tampering** | Alteración de los logs de pacientes durante el tránsito. | Firmado HMAC de payloads y comunicación cifrada TLS. |
| **Repudiation** | Un usuario niega haber cambiado la prioridad de urgencia. | Audit log inmutable con `trace_id` correlacionado. |
| **Information Disclosure** | Fuga de datos de PII de pacientes en logs de la app. | Redacción de PII con Ollama Llama 3.2 local antes de persistir. |
| **Denial of Service** | Saturación de la API de admisión de emergencias. | Rate limiting en Ingress Nginx y autoscaling con HPA. |
| **Elevation of Privilege** | Escalado a permisos de administrador en el backend. | RBAC estricto y principio de mínimo privilegio en K8s. |
