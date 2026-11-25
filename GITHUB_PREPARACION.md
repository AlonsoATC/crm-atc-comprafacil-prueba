# CRM ATC Compra Fácil USA · copia para GitHub

Versión de la aplicación: V9.5. Fuente: `db627d228aab859c1dc531ee9f942adf35a6164f`.

Este paquete contiene el código fuente, documentación, esquema y migraciones de D1, recursos de la interfaz y pruebas. Las muestras de prueba son ficticias. Se excluyen la base de datos operativa, credenciales, archivos de entorno, dependencias instaladas, resultados de compilación y capturas operativas. La revisión de patrones sensibles solo identificó una URL de prueba deliberadamente inválida en `tests/documents.test.js`; no es una credencial de servicio.

## Repositorio de destino

Subir a un repositorio **privado** de la organización que el propietario indique. El acceso efectivo depende de los permisos de esa organización. No se configura GitHub Pages ni despliegue automático. Esta copia no altera el sitio privado actual.

La configuración `.openai/hosting.json` conserva la referencia técnica del sitio actual; no contiene credenciales. Antes de desplegar desde otro entorno, decidir expresamente si se actualizará ese sitio o se creará una copia independiente. Los cambios de prueba pueden trabajarse en ramas y revisarse antes de incorporarlos al proyecto principal.

## Código y pruebas

El proyecto utiliza Node 22 o posterior compatible con sus dependencias. Con Node y npm instalados, ejecutar desde la raíz:

```sh
npm ci
npm test
npm run typecheck
npm run build
```

Consultar `README.md` para la operación, y `docs/INTEGRACION_CRM_ATC.md` para la futura fusión con el CRM central. Las credenciales y la base de datos se configuran por separado en el entorno de ejecución; no se incluyen en GitHub. Los datos y el historial del sitio permanecen en su almacenamiento privado.

La regla de DNI continúa siendo una solicitud expresa de Compras por OC y producto. No se deriva del aviso genérico de la base central. Se conserva el flujo de garantías, cierre por agente, trazabilidad multianual y navegación de V9.5.

Canal rojo: acceso directo y registro manual por OC; ver docs/CANAL_ROJO_V9_5.md.
