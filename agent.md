# Role
Eres un experto en product requirement document, crea un documento con todos los detalle de la app, para el frontend apoyate de un UX designer para que la experiencia de usuario sea la mejor, recuerda que la aplicación será utilizada por personas de todas las edades y niveles de conocimiento de tecnología, por tanto debe ser muy intuitiva y fácil de usar. los flujos de trabajo deben estar bien definidos y documentados.

# Requerimientos
La idea es crear una aplicación para administrar conjuntos residenciales (llamada Rexi) con los siguientes requisitos: La aplicación tendrá conjuntos, torres y propietarios. Un propietario puede tener más de 1 apartamento. Los conjuntos se le debe crear las torres y apartamentos, cada apartamento tiene un propietario y los propietarios pueden tener arrendatarios, hay apartamentos que tiene un parqueadero asignado y otros son comunales. El conjunto tiene un prusupuesto anual que sirve para calcular la cuota de los apartamentos a partir del valor del coeficiente propiedad horizontal, si tiene parquedero asignado o deposito tambien tienen un coeficiente que se suma en al apartamento para calcular la administración del apartamento, este calculo es: Cuota de Administración = ((CoeficienteApartamento + CoeficienteParqueadero + CoeficienteDeposito) * Presupuesto Total de Gastos), hay otra opción que la cuota sea fija para todos los apartamentos. 
Los propietarios o arrendatarios deben registrarse en la aplicacion con datos basicos de la familia que vive y si tienen vehiculos suministrar la placa e igual al registrar propietario generar usuario y contraseña, el propietario es el responsable de registrar al arrendatario y generar su usuario a partir del email. En el conjuntos hay zonas comunes (gym, piscina, salones sociales, etc) algunas zonas comunes para utilizarla se requiere un valor de canon otras no. Cada conjunto tiene sus normas de convivencias y otros documentos, tener un bot que pueda responder cualquier pregunta acerca de estos documentos para conocimiento de los propietarios y arrendatarios. Crear consulta para buscar información de arrendatario o propietarios o a partir de un apartamento, crear notificaciones al recibir paquete, donde se le envia notificación al propietario el paquete recibido. 
Buscar por placa si esta registrada el vehiculo en el conjunto con la información del propietario o arrendatario y si tienen autorización de ingreso. 
Notificaciones de información general del conjunto para los propietarios y arrendatarios, se puede enviar por correo, por la aplicación o por la web. 
Notificar a los propietarios cuando se esta por vencer el plazo de pago de la administración y generar historial de pago e igual generar interes de mora para metrizado.
Crea crud para insertar los datos de los conjuntos, torres, apartamentos, propietarios, arrendatarios, zonas comunes y los que veas necesario. 
Genera un dashboard para el administrador del conjunto con los datos mas relevantes. 
Genera informes para el administrador del conjunto con los datos mas relevantes de pagos o información de los propietarios y arrendatarios. 
Añade las automatizaciones que veas necesarias para mejorar la experiencia del usuario. 
Crear con IA una forma de generar reportes a policia, bomberos, ambulancia, Servicio de Energia, Agua, Gas, etc, a sus lineas de atención ya sea con un bot con un chat donde el residente pueda contar lo sucedido y con ia se formule el reporte a las entidades respectivas. 
Los vigilantes pueden generar multas si no estan cumpliendo con las normas los propietarios o arrendatarios, pero la administración debe aprobar la multa, el propietario o arrendatario puede apelar la multa, el administrador puede aprobar o rechazar la apelación, estas multas seran cobradas en la cuota de administración. El administrador debe tener reportes por rango de fechas con totales a pagar y pagados, mora, etc. 
Generar recibo de pago de la administración en pdf y enviarlo al correo del propietario.

Opción para reportar datos en el conjunto y que la inteligencia artificial que pueda categorizar la prioridad para informar a la administración, e igual crear los reportes manuales. 
Crear cargues masivos para los apartamentos por torre. Tambien podra crear visitantes y asignarles un codigo QR o un para que puedan ingresar al conjunto. Los visitantes pueden ser temporales o permanentes, los temporales tienen una fecha de expiración. Los vehiculos de los visitantes deben ser registrados y para poder ser mapeado por la placa y tendrá toda la información del visitante. Los vigilantes podran buscar por placa la información del vehiculo y del propietario o arrendatario. Los Vigilantes podran buscar por el QR o Identificación del visitante. Los propietarios y arrendatarios pueden ingresar su rostro facial para el reconocimiento facial en la entrada del conjunto (opcional). Los vigilantes se visualizará la información del propietario o arrendatario al ingresar al conjunto e igual con los vehiculos. En los listar usar paginación y filtros. 
Incluir interes de mora parametrizable para la cuota administración, se puede configurar el porcentaje o un valor fijo por dia de mora, o por dia de mora. El sistema debe calcular la fecha de vencimiento y la fecha actual y aplicar el interes de mora.
Incluir agregar cuota extraordinaria para una fecha expecifica y agregarlo a la cuota de administración en esa fecha.
Incluir Reportes en base de los recaudos de administración y gastos comunes por mes, para la asamblea, reporte que sirva de base para generar los estados financieros del conjunto.

# Investigación 
Por qué Digitalizar la Administración?
Problemas de la gestión manual Errores en cálculos de cuotas Documentación desordenada o perdida
Comunicación ineficiente Cobro de cuotas difícil de rastrear Conflictos por falta de transparencia Beneficios de un software especializado Automatización de tareas repetitivas
Información centralizada y accesible Transparencia financiera total Comunicación instantánea
Reportes automáticos

Funcionalidades Esenciales
1. Gestión de Pagos
Debe incluir:

Generación automática de cuotas
Múltiples métodos de pago
Recibos digitales
Control de morosidad
Reportes de cobranza
Conciliación bancaria

2. Comunicación con Residentes
Debe incluir:

Notificaciones push y email
Tablero de anuncios digital
Encuestas y votaciones
Chat o mensajería
Directorio de contactos.

3. Reserva de Áreas Comunes
Debe incluir:

Calendario de disponibilidad
Reservas online 24/7
Confirmaciones automáticas
Gestión de pagos de reservas
Límites y reglas configurables

4. Gestión de Mantenimiento
Debe incluir:

Solicitudes de mantenimiento
Seguimiento de tickets
Asignación a proveedores
Historial de intervenciones
Programación de mantenimiento preventivo

5. Administración Financiera
Debe incluir:

Presupuestos
Control de gastos
Estados financieros
Fondo de reserva
Reportes para asamblea

6. Control de Accesos
Deseable:

Registro de visitantes
Códigos QR de acceso
Autorización de entregas
Historial de ingresos

Criterios de Selección
Facilidad de uso
Interfaz intuitiva
Curva de aprendizaje corta
Soporte en español
Capacitación incluida
Adaptabilidad
Configuración personalizable
Escalable según crecimiento
Integraciones disponibles
Actualizaciones frecuentes
Soporte técnico
Disponibilidad (idealmente 24/7)
Canales de atención (chat, email, teléfono)
Tiempo de respuesta
Base de conocimientos
Seguridad
Encriptación de datos
Backups automáticos
Cumplimiento de normativas de privacidad
Autenticación segura

# Propuesta Técnica de Implementación
Basado en la investigación de funcionalidades esenciales y criterios de selección, se propone el siguiente stack de librerías y servicios para garantizar un MVP de alta calidad, bajo costo y escalabilidad:

| Módulo | Funcionalidad | Librería / Servicio | Razón |
| :--- | :--- | :--- | :--- |
| **Finanzas** | Automatización de Cuotas | **Hangfire** | Manejo robusto de tareas en segundo plano con reintentos automáticos. |
| **Pagos** | Pasarela Online (PSE) | **Wompi / PayU** | Integración nativa con medios de pago colombianos. |
| **Documentos** | Recibos en PDF | **QuestPDF** | Generación de reportes PDF ultra rápida y escalable. |
| **Seguridad** | Códigos QR | **QRCoder** | Librería ligera para generación offline de QR. |
| **IA / Bot** | Chat RAG | **Azure OpenAI + pgvector** | Inteligencia artificial de nivel empresarial con datos privados del conjunto. |
| **IA / Reportes** | Triage de Emergencias | **Azure OpenAI** | Formulación de reportes estructurados para entidades de socorro. |
| **Acceso** | Reconocimiento Facial | **Azure AI Face** | Servicio administrado con alta precisión y seguridad de datos. |
| **Notificaciones** | Push & Email | **FCM + SendGrid** | Estándares de la industria para comunicación masiva y confiable. |
| **Tiempo Real** | Alertas de Portería | **SignalR** | Notificaciones instantáneas para residentes y guardas. |

Esta selección prioriza librerías con fuerte soporte en .NET y servicios de Azure que permiten un crecimiento controlado (Pay-as-you-go).

---

# Flujos de Trabajo
Crea los flujos de trabajo para la aplicación, ten en cuenta que la aplicación será utilizada por personas de todas las edades y niveles de conocimiento de tecnología, por tanto debe ser muy intuitiva y fácil de usar. los flujos de trabajo deben estar bien definidos y documentados.

# Frontend
Genera una propuesta para la arquitectura del frontend, basado en react (puede ser con nextjs o vite), debe tener una arquitectura modular con servicios, componentes, etc,  tener en cuenta costos bajos de mantenimiento y escalabilidad. Proyección de crecimiento de 100 conjuntos en el primer año.

# Backend
Genera una propuesta para la arquitectura del backend, basada en net 10 o superior, utilizar Entity Framework Core para el ORM, clean architecture, repository pattern, unit of work, mediatR y CQRS. tener en cuenta costos bajos de mantenimiento y escalabilidad. Opta por una arquitectura monolitico modular y escalable con capacidad de evolucionar a microservicios si es necesario.  

# Base de Datos
Genera una propuesta para la arquitectura de la base de datos, basada en postgresql, separando por cada modulo del backend en un schema para base de datos y proyectar a una separación de microservicios en el futuro.

# Tareas
Crea un documento con el detalle de cada modulo, con sus respectivas tablas, campos, relaciones, flujos de usuario, casos de uso, pruebas unitarias y de integracion, diagramas de flujo, diagramas de secuencia, diagramas de clases, diagramas de casos de uso, diagramas de actividades, diagramas de componentes, diagramas de despliegue, diagramas de estados, diagramas de paquetes, diagramas de perfil, diagramas de objetos, diagramas de interacción, diagramas de colaboración, diagramas de componentes, diagramas de despliegue, diagramas de estados, diagramas de paquetes, diagramas de perfil, diagramas de objetos, diagramas de interacción, diagramas de colaboración. Este documento sera para el equipo de desarrollo. Crea una carpeta donde en la carpeta esten varios archivos de las tareas que se deben realizar, desde la implementación inicial del proyecto hasta las requerimiento a desarrollar, 1 requerimiento por archivo depende tambien de las fases pertenezca ese requerimiento, detalla lo minimo necesario para que el equipo de desarrollo pueda entender y desarrollar el requerimiento. Los requerimientos separarlos por carpeta para el backend y frontend. 
- Debes detallar el campo de cada ventana y sus validaciones.
- Crea una tarea para construir el modelo EDR y las tablas de la base de datos.

# Consideraciones
- Crear el modelo de base de datos y codigo backend, frontend en ingles.
- Tener en cuenta la aplicación frontend soporte multidiomas. Español e Ingles.
- Crear docker para cada aplicacion y asi facil para su despliegue en azure.
- Genera muy detallado estos requerimientos y escoge cuales pueden ser los escenciales para crear un MVP.
- Crea toda esta documentación en una carpeta /docs en formato markdown.

# UI/UX
En una carpeta /ux, crea mockups para cada una de las ventanas de la aplicación. ten en cuenta que la aplicación frontend soporte multidiomas. Español e Ingles. Usa las mejores practicas de UI/UX y que sea responsive para todas las pantallas (Desktop, Tablet, Mobile). los mockups deben estar en formato html y puedes utilizar css, tailwindcss, bootstrap 5, etc, sin importar el lenguaje de programacion que se vaya a usar en el frontend.


No permitir eliminar este archivo.