# ADR 0006: Política de Firma de Imágenes, Attestations SLSA y Seguridad en Runtime

## Estado
Aceptado

## Contexto
Para proteger la cadena de suministro de software del microservicio `meditriage-backend` frente a compromisos en el pipeline de CI/CD o despliegues de artefactos manipulados, se requiere garantizar la integridad y procedencia criptográfica desde la construcción hasta la admisión en Kubernetes.

## Decisión
1. **Firma Keyless con Cosign (Sigstore):** Todas las imágenes de contenedor publicadas en GHCR se firmarán utilizando identidad OIDC emitida por GitHub Actions (`token.actions.githubusercontent.com`), eliminando el riesgo de gestión y rotación de llaves privadas estáticas.
2. **Attestation SLSA Nivel 2:** Se adjuntará un predicado de procedencia (`slsaprovenance`) en el registro OCI verificable mediante `cosign verify`.
3. **Control de Admisión (Kyverno):** Se implementa la política `require-image-signature` en modo `Enforce` sobre el namespace `meditriage-prod` para rechazar cualquier Pod cuya imagen no posea una firma válida emitida por el workflow oficial.
4. **Detección en Runtime (Falco):** Se definen 3 reglas personalizadas para alertar sobre ejecución de shells interactivas, escritura en `/etc` y conexiones salientes anómalas hacia canales de respuesta a incidentes[cite: 15].

## Consecuencias
* **Positivas:** Trazabilidad criptográfica completa (auditable) y bloqueo preventivo de imágenes no autorizadas en el clúster[cite: 15].
* **Negativas:** Añade ~15-20 segundos de latencia al pipeline de CI y requiere que el webhook de Kyverno esté disponible en el clúster.
