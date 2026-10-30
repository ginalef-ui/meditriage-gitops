# ADR 0005: Estrategia de Despliegue con GitOps y Argo CD

## Estado
Aceptado

## Contexto
MediTriage gestiona datos de salud sensibles y de carácter crítico regulados por normativas legales (Ley 19.628 y Ley 21.719), con requerimientos de disponibilidad del 99.5% y estricta trazabilidad de auditoría. Anteriormente, el despliegue requería la aplicación directa o manual de manifiestos (`kubectl apply`), lo que introducía riesgos de desincronización silenciosa del estado del clúster (*configuration drift*) y forzaba la exposición de credenciales administrativas del clúster dentro de herramientas externas de CI/CD.

## Decisión
Adoptar el modelo de reconciliación **Pull-based (GitOps)** utilizando **Argo CD** como operador dentro del clúster y **GitHub Container Registry (GHCR)** como registro de artefactos:

1. **Separación de repositorios:** Se mantiene el código fuente y el pipeline de CI en el repositorio de la aplicación (`meditriage`), y los manifiestos declarativos organizados con Kustomize (`base/`, `overlays/dev/`, `overlays/prod/`) en un repositorio dedicado (`meditriage-gitops`).
2. **Reconciliación continua:** Argo CD se instala dentro del clúster local/dev y monitorea continuamente la rama principal del repositorio GitOps, aplicando automáticamente cualquier cambio aprobado.
3. **Pipeline desacoplado:** El pipeline de GitHub Actions se limita a tareas de integración continua (calidad, pruebas, escaneo SAST, construcción y publicación de la imagen en GHCR), sin requerir acceso ni llaves directas al clúster de Kubernetes.

## Alternativas descartadas
* **Estrategia Push-based (CI clásico con GitHub Actions):** Descartada porque requiere almacenar tokens y credenciales de acceso directo del clúster dentro de GitHub Secrets, aumenta la superficie de ataque y no previene desviaciones si el clúster es modificado externamente.
* **Flux CD:** Aunque es ligero y modular, se descartó en favor de Argo CD debido a la necesidad del equipo de contar con una interfaz gráfica (UI) completa para observabilidad en tiempo real, control de acceso basado en roles (RBAC) y soporte nativo para despliegues progresivos (Canary con Argo Rollouts).

## Consecuencias
* **Positivas:**
  * **Seguridad (Zero-Trust):** Ninguna credencial de infraestructura ni del clúster reside fuera del entorno de ejecución; Argo CD extrae los cambios desde el interior de la red.
  * **Trazabilidad y Cumplimiento:** Todo cambio de versión o configuración queda auditado mediante Pull Requests aprobados en Git, cumpliendo los requerimientos regulatorios.
  * **Detección y reparación de drift:** Cualquier alteración manual no autorizada en el clúster es detectada y revertida automáticamente a la verdad declarada en Git.
  * **Rollbacks instantáneos:** Revertir un despliegue en producción se reduce a revertir un commit (`git revert`) en el repositorio GitOps.
* **Negativas:**
  * Introduce sobrecarga operativa al requerir la sincronización y mantenimiento de dos repositorios separados.
  * Dependencia del ciclo de vida y consumo de recursos del operador de Argo CD dentro del clúster.
```[cite: 2, 10]

