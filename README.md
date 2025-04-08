# Eliminar Advertencias al Enviar Emails desde Gmail a Outlook con Dominio Personalizado

Este documento explica cómo resolver el problema de las advertencias de seguridad que aparecen cuando envías correos electrónicos desde un dominio personalizado configurado en Cloudflare a cuentas de Outlook.

## El Problema

Cuando envías correos electrónicos desde un dominio personalizado (ejemplo@tudominio.com) configurado a través de servicios como Gmail o Google Workspace, y estos correos son recibidos por direcciones de Outlook, es común que aparezcan advertencias como:

- "Este remitente no ha sido verificado"
- "El mensaje podría ser una estafa"
- Banners de advertencia rojos o amarillos

Esto ocurre porque Outlook implementa estrictas verificaciones de autenticación de correo electrónico.

## Solución: Configuración Correcta de Registros DNS

Para eliminar estas advertencias es necesario configurar correctamente tres tipos de registros DNS en Cloudflare:

### 1. Configurar SPF (Sender Policy Framework)

El registro SPF especifica qué servidores están autorizados a enviar correos desde tu dominio.

#### Pasos en Cloudflare:
1. Inicia sesión en tu panel de Cloudflare
2. Selecciona tu dominio
3. Ve a la sección "DNS"
4. Añade un registro TXT con las siguientes características:
   - **Tipo**: TXT
   - **Nombre**: @ (o deja en blanco)
   - **Contenido**: `v=spf1 include:_spf.google.com ~all`
   - **TTL**: Auto

> **Nota**: Si ya tienes otros servicios enviando correos desde tu dominio, asegúrate de incluirlos en el registro SPF.

### 2. Configurar DKIM (DomainKeys Identified Mail)

DKIM añade una firma digital cifrada a tus correos electrónicos.

#### Pasos en Google Workspace/Gmail:
1. Inicia sesión en la consola de administración de Google Workspace
2. Ve a "Apps" > "Google Workspace" > "Gmail" > "Autenticación de correo electrónico"
3. Haz clic en "Generar nuevo registro" para DKIM
4. Selecciona tu dominio y tamaño de clave (recomendado: 2048 bits)
5. Google te proporcionará la información del registro CNAME que debes crear

#### Pasos en Cloudflare:
1. Añade un registro CNAME con las características proporcionadas por Google:
   - **Tipo**: CNAME
   - **Nombre**: [selector proporcionado por Google]._domainkey
   - **Destino**: [valor proporcionado por Google]
   - **TTL**: Auto

### 3. Configurar DMARC (Domain-based Message Authentication, Reporting & Conformance)

DMARC define qué hacer cuando un correo no pasa las verificaciones SPF o DKIM.

#### Pasos en Cloudflare:
1. Añade un registro TXT con las siguientes características:
   - **Tipo**: TXT
   - **Nombre**: _dmarc
   - **Contenido**: `v=DMARC1; p=quarantine; rua=mailto:tu_email@tudominio.com; pct=100; adkim=s; aspf=s`
   - **TTL**: Auto

> **Nota**: 
> - `p=quarantine` indica que los correos sospechosos se envíen a spam. Puedes usar `p=none` para empezar (solo monitoreo).
> - `rua=mailto:tu_email@tudominio.com` especifica dónde enviar informes de cumplimiento (cambia por tu email).

## Verificación

Para verificar que tus configuraciones están correctas:
1. Usa la herramienta [MX Toolbox](https://mxtoolbox.com/SuperTool.aspx) para comprobar tus registros SPF, DKIM y DMARC
2. Envía un correo de prueba a una dirección de Outlook y verifica que no aparecen advertencias

## Tiempos de Propagación

Los cambios en registros DNS pueden tardar hasta 48 horas en propagarse completamente. Sé paciente si las advertencias no desaparecen inmediatamente.

## Problemas Frecuentes

- **La advertencia persiste**: Verifica que los tres registros (SPF, DKIM, DMARC) están correctamente configurados
- **Problemas con subdominios**: Asegúrate de configurar registros para cualquier subdominio que utilices para enviar correos
- **Correos marcados como spam**: Comienza con una política DMARC menos restrictiva (`p=none`) y gradualmente aumenta la seguridad

## Recursos Adicionales

- [Documentación de Cloudflare sobre Email Security](https://developers.cloudflare.com/email-security/)
- [Guía de Google Workspace para autenticación de correo electrónico](https://support.google.com/a/answer/174124)
- [Verificador DMARC](https://dmarcian.com/dmarc-inspector/)
- [Verificador SPF](https://www.spfwizard.net/)

---

Si sigues estos pasos deberías poder eliminar las advertencias de seguridad al enviar correos desde tu dominio personalizado configurado en Cloudflare a direcciones de Outlook.