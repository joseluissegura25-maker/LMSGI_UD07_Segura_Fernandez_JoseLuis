# Manual Técnico de Explotación - WillmanTech S.L.

Este documento técnico servirá de guía para los administradores de sistemas y usuarios finales que exploten el ERP de "WillmanTech S.L.". Ha sido estructurado siguiendo los requisitos de calidad y usabilidad del estándar internacional de ingeniería de software ISO/IEC/IEEE. 

## 1. Introducción y Arquitectura

* La infraestructura ERP/CRM de WillmanTech S.L. ha sido implantada con el objetivo de optimizar sus flujos de ventas y facturación.
* Para asegurar un funcionamiento aislado y escalable, el sistema informático se encuentra desplegado en un entorno dockerizado.
* La topología lógica se implementa mediante un despliegue con Docker Compose, donde se habilitan y conectan los módulos de aplicación (ERP/CRM) con sus respectivos servicios de persistencia de datos.

## 2. Guía de Instalación y Reinstalación

Para levantar el entorno desde cero y asegurar la correcta inicialización del sistema, se deben ejecutar los siguientes pasos:

* **Clonación del repositorio:** Descargar el código fuente de la infraestructura en el servidor destino.
* **Configuración de Variables de Entorno:** Es estrictamente necesario definir las variables de entorno operativas (ej. credenciales, puertos y rutas) en un archivo `.env` en la raíz del proyecto.
* **Dependencias del SGBD:** El Sistema Gestor de Bases de Datos relacional debe aprovisionarse adecuadamente a través de su propia imagen de contenedor.
* **Despliegue:** Ejecutar el comando `docker-compose up -d` para construir y levantar tanto la base de datos como el servicio web del ERP de forma orquestada.

## 3. Seguridad y Control de Acceso

El control de los privilegios sobre la información se gestiona a través de una matriz de accesos basada en roles específicos y políticas de contraseñas robustas:

* **Rol Administrador:** Cuenta con permisos absolutos de lectura, escritura y ejecución sobre todos los módulos del sistema, así como la gestión de configuraciones globales y mantenimiento de usuarios.
* **Rol Contable:** Posee privilegios operativos centrados en los módulos financieros, permitiendo gestionar la facturación, exportación de transacciones e informes fiscales.
* **Rol Comercial:** Dispone de un acceso delimitado al CRM y procesos de ventas, permitiendo la interacción con clientes y presupuestos, pero restringiendo la manipulación contable o de configuraciones del sistema.

## 4. Procedimiento de Backup y Restauración

La política de copias de seguridad exige respaldar de forma regular la persistencia del sistema:

* **Backup de la base de datos relacional:** Se debe ejecutar una herramienta de volcado del SGBD a través de un comando (ej. `docker exec -t [contenedor_db] pg_dump -U [usuario] [nombre_db] > backup.sql`) para extraer toda la información estructurada.
* **Backup de los almacenes de datos asociados:** Es obligatorio comprimir y salvaguardar de manera periódica los volúmenes Docker (filestore) donde el sistema aloja los archivos físicos, imágenes y adjuntos.
* **Restauración:** En caso de desastre, el proceso inverso requiere levantar los contenedores en un entorno limpio, importar el volcado `.sql` a la nueva base de datos y restaurar la carpeta de volúmenes en su ruta de persistencia asignada.

## 5. Flujo Operativo de Facturación e Informes

El proceso de emisión de facturas y la generación de sus respectivos documentos PDF sigue este ciclo de vida:

* **Gestión en Interfaz:** Un usuario (ej. Contable) consolida un pedido, genera una factura desde la interfaz del ERP y la valida en el sistema.
* **Procesamiento de Plantillas:** Al solicitar la impresión, el sistema toma los datos de la factura y los inyecta dinámicamente en plantillas personalizadas basadas en lenguajes de marcas lógicos (sintaxis QWeb).
* **Pipeline de Renderizado:** El motor del ERP procesa estas plantillas (como el archivo `report_invoice_willmantech.xml`) y renderiza un documento `HTML` temporal.
* **Generación del Documento Final:** Finalmente, la utilidad interna intercepta ese código web y utiliza el motor `wkhtmltopdf` para transcodificar fielmente el documento `HTML` y entregarlo como un archivo `PDF` descargable.