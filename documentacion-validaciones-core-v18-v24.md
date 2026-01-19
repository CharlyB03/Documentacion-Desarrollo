# Documentacion Validaciones Core V18 - V24

Se realizó un análisis de los cambios aplicados en el Core de Openbravo de la versión 18 y su comparación con la versión 25, con el fin de evaluar si la integración es óptima. Este análisis se basó en las modificaciones detalladas en cada ticket, verificando primero los cambios implementados y posteriormente evaluando cada parte del codigo y sus respectivos detalles.

### Ticket - 2042 COMPRAS - DESARROLLO - Añadir nuevas columnas en Reportes y Mayor"

Se identificaron **cambios en los reportes** ubicados en la ruta **src/org/openbravo/erpCommon/ad\_reports/ReportGeneralLedgerJournal.jrxml**, los cuales fueron analizados como parte de la revisión de modificaciones realizadas sobre el Core de Openbravo.

```
<staticText>
				<reportElement key="element-90" style="Detail_Header" x="350" y="45" width="150" height="15" uuid="54ab7196-4a47-4e90-a002-63074110a09c">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitwidth" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
				</reportElement>
				<box leftPadding="5" rightPadding="2">
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#333333"/>
				</box>
				<textElement>
					<font fontName="SansSerif" size="8"/>
				</textElement>
				<text><![CDATA[Referencia]]></text>
			</staticText>
```

```
</textField>
			<textField pattern="dd/MM/yyyy" isBlankWhenNull="true">
				<reportElement key="textField-22" stretchType="RelativeToBandHeight" x="350" y="0" width="150" height="13" uuid="3ea97745-cb2a-4823-99ca-d4bcedba230c">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box rightPadding="2">
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
				</box>
				<textElement textAlignment="Left">
					<font fontName="SansSerif" size="8"/>
				</textElement>
				<textFieldExpression><![CDATA[$F{DOCUMENTNO}]]></textFieldExpression>
			</textField>
		</band>
```

Se tomó en cuenta lo siguiente:&#x20;

Se **crea visualmente la columna “Referencia” en el encabezado del reporte de Libro Mayor**, lo cual **permite al usuario identificar las facturas mediante su número de documento directamente desde el reporte**, evitando búsquedas manuales.&#x20;

Esta modificación es **óptima para su integración**, ya que **mejora la productividad del usuario** y **no afecta el rendimiento del ambiente AgilERP24Q4.1**.



### Ticket  -  Control Logístico de Despachos Parciales y Validación de Cantidades (Tickets: 23500, 24116, 24179)

Se observó que en la versión 18 se añaden filtros y operaciones matemáticas complejas que realizan consultas en tiempo real sobre todas las facturas y devoluciones relacionadas con un pedido, con el objetivo de determinar la cantidad de producto disponible. Todo este procesamiento se ejecuta a nivel de base de datos, donde es posible identificar y analizar las consultas adicionales incorporadas.

El código identificado aportó de las siguientes formas clave:

* Evitó inconsistencias en inventarios, impidiendo que los usuarios despachen cantidades superiores a las registradas en el pedido original, reduciendo así errores por duplicidad o fallas humanas.
* Sincronizó correctamente las devoluciones, permitiendo que el sistema registre automáticamente la mercadería devuelta y la considere nuevamente disponible para facturación o despacho, sin requerir procesos manuales adicionales.
* Permitió la gestión de entregas parciales, facilitando el despacho progresivo de la mercadería y manteniendo un control preciso sobre las cantidades pendientes por entregar.

¿Es óptimo implementarlo en la nueva versión?\
Sí, es indispensable. Aunque la nueva versión cuenta con una arquitectura más moderna, no incluye estas reglas de negocio específicas. Si no se integran, el sistema permitirá que los usuarios realicen errores en los procesos de despacho y facturación, generando duplicidades que ya se encontraban controladas en la versión anterior. Por lo tanto, su implementación es un paso obligatorio para mantener la calidad y consistencia de la operación logística.



### Ticket 17640 -  Creación de atributo automatico DIA JULIANO

Se puede observar que en la versión 18 se encuentra implementada la funcionalidad que permite asignar de manera automática el atributo correspondiente al día juliano.

```java
  
    if (fields[0].emCsljJuliandate.equals("Y")) {
    	
    	
        strHtml.append("<tr><td class=\"TitleCell\"><span class=\"LabelText\">");
        
        LocalDate today = LocalDate.now();  // Fecha actual
        int julianDay = today.getDayOfYear();  // Día juliano del año
        int year = LocalDate.now().getYear();
        int lastTwoDigits = year % 100;

        
        String strNameCell = Utility.messageBD(this, "SerNo", vars.getLanguage());
        String strName = String.format("%02d%03d", lastTwoDigits, julianDay);
        strHtml.append(strNameCell.equals("") ? "emCsljJuliandate" : "DÍA JULIANO");
        strHtml.append("</span></td>\n");
        strHtml.append("<td class=\"");
        if (fields[0].isoneattrsetvalrequired.equals("Y")) {
          strHtml.append("Key");
        }
        strHtml.append("TextBox_ContentCell\"><input type=\"text\" ");
        strHtml.append("name=\"inpDateJulianxxxinternalOB\" ");
        strHtml.append("maxlength=\"20\" ");
        strHtml.append("class=\"dojoValidateValid TextBox_OneCell_width");
        if (fields[0].isoneattrsetvalrequired.equals("Y")) {
          strHtml.append(" required\"  required=\"true\"");
        } else {
            strHtml.append("\"");
        }
      
        // Asignar el valor (editable siempre)
        strHtml.append("value=\"" + strName + "\" ");

        strHtml.append("></td><td></td><td></td></tr>\n");
      }
    
```

Este código añade una funcionalidad automática en la pantalla de creación de lotes o atributos de productos. La lógica implementada genera un código de forma automática tomando como base el año actual en formato abreviado (por ejemplo, 26 para el año 2026) y el número correspondiente al día juliano del año (del 1 al 365), construyendo así un identificador único, como 26016 para el 16 de enero de 2026.

**Integración a la nueva versión**\
Sí, es óptima, pero requiere una configuración previa. Para que esta funcionalidad opere correctamente en la nueva versión, es necesario verificar que en la base de datos de AgilERP24Q4.1 exista la columna `em_cslj_juliandate` en la tabla de atributos. En caso de que dicha columna no se encuentre creada, el sistema generará un error al intentar acceder a la pantalla correspondiente.

### Ticket 23419/23419 - GLPI 2896 ERROR ENVIO DE ROLES

La versión 18 incluye un parche de seguridad orientado a forzar la conexión con servidores de correo antiguos o mal configurados. En contraste, la versión 24 utiliza un motor de correo más moderno, el cual soporta autenticación OAuth2 para servicios como Gmail y Outlook, y realiza el envío de correos en segundo plano, evitando bloqueos en la interfaz del usuario. Por lo tanto, se recomienda mantener la implementación de la versión 24, ya que ofrece un mayor nivel de seguridad, mejor compatibilidad y una arquitectura más actualizada.

### Ticket 1860  - "DESARROLLO - QUITAR ENCABEZADO DE NORMA ISO EN REPORTES"

Dentro de este ticket se identificó el siguiente código, cuya función es solicitar al sistema que envíe al reporte la información del usuario que ha ejecutado la acción al hacer clic en el botón de **“Imprimir”**, permitiendo así registrar y mostrar los datos de la persona que generó el documento.

```
<parameter name="User_ID" class="java.lang.String" isForPrompting="false">
		<defaultValueExpression><![CDATA[OBContext.getOBContext().getUser().getId()]]></defaultValueExpression>
	</parameter>
	<parameter name="UserName" class="java.lang.String" isForPrompting="false">
		<defaultValueExpression><![CDATA[OBContext.getOBContext().getUser().getUsername()]]></defaultValueExpression>
	</parameter>
```

La versión 18 contaba con etiquetas traducidas al español y un encabezado corporativo que incluía el logotipo institucional y los datos del usuario, mientras que la versión nueva se presenta de forma genérica y en idioma inglés

Esto permite que el área de compras trabaje con términos familiares y que los documentos que salen de la empresa tengan el logo oficial y control de quién los emitió.

¿Es óptimo integrarlo? Sí. Es necesario trasladar las etiquetas en español y el diseño del encabezado de la versión 18 a la versión 24, a fin de preservar la identidad visual y la facilidad de uso del sistema.



### **Ticket 4042 - "DESA\_4042\_ERROR EN CARGA DE REPORTES UNNO**

En este caso se decidió mantener la versión v24, ya que corresponde a una actualización técnica necesaria. La versión 18 utiliza instrucciones de diseño obsoletas que no son compatibles con el motor de impresión actual, lo que podría provocar que los nombres de los productos se muestren incompletos o que el reporte genere errores al momento de su ejecución. La versión v24, en cambio, ya incorpora el lenguaje y las correcciones requeridas por el nuevo motor de impresión, lo que la hace más estable, segura y adecuada para el entorno AgilERP24Q4.1.



### Ticket  7763 - Reporte de transferencias internas

Se identificó esta diferencia en la versión 18. Este fragmento de código habilita la función `submitCommandForm('EXCEL', ...)`, la cual permite la exportación de los movimientos de inventario. En ausencia de este bloque dentro del archivo `.html`, el usuario únicamente puede visualizar la información en la interfaz web (HTML). Al incorporarlo, se restablece la posibilidad para las áreas de contabilidad o bodega de descargar los movimientos en formato Excel, facilitando la realización de cruces de información, filtros manuales y la generación de reportes rápidos.

```
              <td class="Button_LeftAlign_ContentCell" colspan="0">
                <div>
              	  <button type="button" 
                    id="buttonHTML" 
                    class="ButtonLink" 
                    onclick="checkEmptyFields();submitCommandForm('EXCEL', true, null, 'ReportProductMovement.html', '_self', null, false);return false;" 
                    onfocus="buttonEvent('onfocus', this); window.status='View Results in a New Window'; return true;" 
                    onblur="buttonEvent('onblur', this);" 
                    onkeyup="buttonEvent('onkeyup', this);" 
                    onkeydown="buttonEvent('onkeydown', this);" 
                    onkeypress="buttonEvent('onkeypress', this);" 
                    onmouseup="buttonEvent('onmouseup', this);" 
                    onmousedown="buttonEvent('onmousedown', this);" 
                    onmouseover="buttonEvent('onmouseover', this); window.status='View Results in a New Window'; return true;" 
                    onmouseout="buttonEvent('onmouseout', this);">
                    <table class="Button">
                      <tr>
                        <td class="Button_left"><img class="Button_Icon Button_Icon_html" alt="View Results in a New Window" title="View Results in a New Window" src="../../../../../web/images/blank.gif" border="0" /></td>
                        <td class="Button_text">EXCEL Format</td>
                        <td class="Button_right"></td>
                      </tr>
                    </table>
                  </button>
                </div>
              </td>    
```

Esta integración presente en la versión 18 resulta de gran utilidad para el área de logística, ya que habilita un botón necesario para cuadrar los movimientos de salida y entrada de mercadería entre bodegas.

Su implementación representa una propuesta adecuada, debido a que permite al responsable de inventarios descargar el reporte correspondiente y comparar de forma rápida, mediante una hoja de cálculo, si las cantidades despachadas desde la bodega “A” coinciden con las recibidas en la bodega “B”, facilitando así la identificación de posibles faltantes o sobrantes.



### Ticket 2804 - "ACTIVOS - DESARROLLO - REFERENCIA EN LIBRO MAYOR Y EXCEL"

Se identificaron dos variaciones en la versión 18.

```
<staticText>
				<reportElement key="element-90" style="Detail_Header" x="350" y="45" width="150" height="15" uuid="54ab7196-4a47-4e90-a002-63074110a09c">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitwidth" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
				</reportElement>
				<box leftPadding="5" rightPadding="2">
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#333333"/>
				</box>
				<textElement>
					<font fontName="SansSerif" size="8"/>
				</textElement>
				<text><![CDATA[Referencia]]></text>
			</staticText>
```

```
<textField pattern="dd/MM/yyyy" isBlankWhenNull="true">
				<reportElement key="textField-22" stretchType="RelativeToBandHeight" x="350" y="0" width="150" height="13" uuid="3ea97745-cb2a-4823-99ca-d4bcedba230c">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box rightPadding="2">
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
				</box>
				<textElement textAlignment="Left">
					<font fontName="SansSerif" size="8"/>
				</textElement>
				<textFieldExpression><![CDATA[$F{DOCUMENTNO}]]></textFieldExpression>
```

Este código obtiene el campo Documento desde la base de datos y lo presenta en el reporte con un formato de texto alineado a la izquierda. Su objetivo es permitir que el usuario identifique de forma rápida el número de factura o asiento contable asociado a cada movimiento. Se trata de un componente clave de la versión 18 que puede mantenerse en la versión 24, ya que permite conservar la trazabilidad de los documentos dentro del reporte de Libro Mayor.



### Ticket 18115 - DESARROLLO DIARIO CONTABLE DE VACACIONES

Dentro del desarrollo se identifica una personalización en la versión 18, mientras que la versión 24 presenta una implementación más genérica y con menor nivel de detalle. Por este motivo, resulta viable adaptar dicha personalización a la versión 24 para mantener el comportamiento funcional esperado.

```
<image ...>
  <imageExpression><![CDATA[
    org.openbravo.erpCommon.utility.Utility.showImageLogo("yourcompanylegal",$F{ad_org_id2})
  ]]></imageExpression>
</image>

```

```
<subreport>
  <reportElement ...>
    <printWhenExpression><![CDATA[
      $F{AD_TABLE_ID}.equals("881B6BC8F33E49168898C1FB4994099F")
    ]]></printWhenExpression>
  </reportElement>
  <subreportParameter name="ID">
    <subreportParameterExpression><![CDATA[$F{ID}]]></subreportParameterExpression>
  </subreportParameter>
  <connectionExpression><![CDATA[$P{REPORT_CONNECTION}]]></connectionExpression>
  <subreportExpression><![CDATA[
    $P{BASE_DESIGN} + "/org/openbravo/erpCommon/ad_reports/ReportGeneralLedgerJournalVacations.jasper"
  ]]></subreportExpression>
</subreport>

```

Este código permite la generación de un reporte de asientos contables (Journal Entries) a partir de información agregada de la tabla FACT\_ACCT, incluyendo la lista de cuentas, su descripción y los valores correspondientes de Debe y Haber. Adicionalmente, al final del reporte se realiza el cálculo de los totales acumulados de Debe y Haber.

Si es óptimo integrar los elementos funcionales de la versión 18 en la versión 24. Se recomienda migrar únicamente aquellos componentes que aportan valor funcional, como el campo de comprobante y el encabezado corporativo con logotipo o subreporte, manteniendo la versión 24 como base debido a su mejor legibilidad, tipado y estructura de agrupación.



### Ticket 1074 - "Mejoras al reporte de Antiguedad de Cartera Cobro y Pagos"

Este reporte clasifica las deudas de clientes o proveedores por rangos de antigüedad, por ejemplo de 1 a 30 días, de 31 a 60 días, entre otros..

* La v18  está diseñada para que el departamento de cobranzas  pueda ver de un vistazo qué facturas están próximas a vencer y cuáles ya están morosas, identificando al cliente por su RUC.
* En cambio, la versión 24 corresponde al formato base de Openbravo, el cual es funcional, pero resulta menos intuitivo para usuarios de habla hispana y para la generación de reportes con fines legales.

```
	<textField isBlankWhenNull="true" hyperlinkType="Reference">
					<reportElement key="textField-6" style="Detail_Header" x="1053" y="5" width="76" height="16" forecolor="#000000" backcolor="#FFFFFF" uuid="137097b2-182b-4fc1-afbe-21743eed1ed2">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
					<box leftPadding="2"/>
					<textElement textAlignment="Right" verticalAlignment="Middle">
						<font size="8" isBold="false" isUnderline="false"/>
					</textElement>
					<textFieldExpression><![CDATA[$V{SUMAMT4} != null ? $P{AMOUNTFORMAT}.format($V{SUMAMT4}) : $P{AMOUNTFORMAT}.format(new BigDecimal("0"))]]></textFieldExpression>
				</textField>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="1062" y="2" width="71" height="1" uuid="6629c116-06bb-4747-aa61-0c09cc0ff92f">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="1062" y="23" width="71" height="1" uuid="b2d97532-a9cd-44aa-917c-f34b4086a2b6">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="1062" y="25" width="71" height="1" uuid="20099c84-8e7a-4f49-a308-0abae0831694">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
				</line>
				<textField isBlankWhenNull="true" hyperlinkType="Reference">
					<reportElement key="textField-6" style="Detail_Header" x="978" y="5" width="74" height="16" forecolor="#000000" backcolor="#FFFFFF" uuid="dd3aebcf-ce17-4221-9f36-78040a114f83">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
					<box leftPadding="2"/>
					<textElement textAlignment="Right" verticalAlignment="Middle">
						<font size="8" isBold="false" isUnderline="false"/>
					</textElement>
					<textFieldExpression><![CDATA[$V{SUMAMT3} != null ? $P{AMOUNTFORMAT}.format($V{SUMAMT3}) : $P{AMOUNTFORMAT}.format(new BigDecimal("0"))]]></textFieldExpression>
				</textField>
				<textField isBlankWhenNull="true" hyperlinkType="Reference">
					<reportElement key="textField-6" style="Detail_Header" x="903" y="5" width="76" height="16" forecolor="#000000" backcolor="#FFFFFF" uuid="cdc225ec-a9ce-4326-a3f2-ea49355c7738">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
					<box leftPadding="2"/>
					<textElement textAlignment="Right" verticalAlignment="Middle">
						<font size="8" isBold="false" isUnderline="false"/>
					</textElement>
					<textFieldExpression><![CDATA[$V{SUMAMT2} != null ? $P{AMOUNTFORMAT}.format($V{SUMAMT2}) : $P{AMOUNTFORMAT}.format(new BigDecimal("0"))]]></textFieldExpression>
				</textField>
				<textField isBlankWhenNull="true" hyperlinkType="Reference">
					<reportElement key="textField-6" style="Detail_Header" x="828" y="5" width="75" height="16" forecolor="#000000" backcolor="#FFFFFF" uuid="1738c5ab-34c5-4cb2-9b4d-0669c013291b">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="local_mesure_unitwidth" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.width" value="px"/>
					</reportElement>
					<box leftPadding="2"/>
					<textElement textAlignment="Right" verticalAlignment="Middle">
						<font size="8" isBold="false" isUnderline="false"/>
					</textElement>
					<textFieldExpression><![CDATA[$V{SUMAMT1} != null ? $P{AMOUNTFORMAT}.format($V{SUMAMT1}) : $P{AMOUNTFORMAT}.format(new BigDecimal("0"))]]></textFieldExpression>
				</textField>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="987" y="2" width="71" height="1" uuid="11c61558-6794-4974-87ce-ea96b91e79b8">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="987" y="23" width="71" height="1" uuid="9ac0234f-2527-4942-881a-368f8a4a467f">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="912" y="2" width="71" height="1" uuid="72bd8137-4795-4643-bbfe-a604a2b2070a">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="837" y="2" width="71" height="1" uuid="f500951b-1b63-49cf-8a2b-02ec4bea296b">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="912" y="23" width="71" height="1" uuid="00bc1b13-370a-4c61-a2b8-d7b152bd4388">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="837" y="23" width="71" height="1" uuid="8d708bd1-04c5-4619-9771-243ed8596cc5">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="837" y="25" width="71" height="1" uuid="56f81d84-d8a2-4d75-ae44-7d353a7610d6">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="912" y="25" width="71" height="1" uuid="e34d7413-d69b-491f-ab3b-d95879f216e1">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="987" y="25" width="71" height="1" uuid="77a22df5-9126-435c-a69b-cae211a78394">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
```

Este código se encarga de generar, al cierre de cada cliente o sección, una fila de totales correctamente alineada y resaltada mediante subrayado. Su lógica realiza la suma de todos los valores correspondientes a la columna, aplica el formato monetario adecuado y agrega elementos visuales que permiten al contador identificar claramente el resultado final de cada grupo de facturas.grupo de facturas.

También se observó lo siguiente:

```
<textField>
				<reportElement key="staticText-25" style="Detail_Header" x="1128" y="30" width="75" height="20" uuid="7531808b-0a44-44aa-95a7-a4e866c92fe9">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<textElement textAlignment="Right" markup="none">
					<font size="8" pdfFontName="Helvetica-Bold"/>
				</textElement>
				<textFieldExpression><![CDATA[$P{inpLabel5}]]></textFieldExpression>
			</textField>
			<textField>
				<reportElement key="staticText-25" style="Detail_Header" x="1053" y="30" width="75" height="20" uuid="ac51d7dc-5b82-4f33-99b7-e57ec9e06bd8">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<textElement textAlignment="Right" markup="none">
					<font size="8" pdfFontName="Helvetica-Bold"/>
				</textElement>
				<textFieldExpression><![CDATA[$P{inpLabel4}]]></textFieldExpression>
			</textField>
			<textField>
				<reportElement key="staticText-25" style="Detail_Header" x="978" y="30" width="75" height="20" uuid="ad824e1e-2846-4eae-a01e-f6d59ec53ab6">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<textElement textAlignment="Right" markup="none">
					<font size="8" pdfFontName="Helvetica-Bold"/>
				</textElement>
				<textFieldExpression><![CDATA[$P{inpLabel3}]]></textFieldExpression>
			</textField>
			<textField>
				<reportElement key="staticText-25" style="Detail_Header" x="903" y="30" width="75" height="20" uuid="030340d5-fb9b-44f6-9d83-3410ddebe519">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<textElement textAlignment="Right" markup="none">
					<font size="8" pdfFontName="Helvetica-Bold"/>
				</textElement>
				<textFieldExpression><![CDATA[$P{inpLabel2}]]></textFieldExpression>
			</textField>
			<textField>
				<reportElement key="staticText-25" style="Detail_Header" x="828" y="30" width="75" height="20" uuid="06d248b7-360d-4874-9a5a-c31ce01d6a57">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<textElement textAlignment="Right" markup="none">
					<font size="8" pdfFontName="Helvetica-Bold"/>
				</textElement>
				<textFieldExpression><![CDATA[$P{inpLabel1}]]></textFieldExpression>
			</textField>
			<textField>
				<reportElement key="staticText-25" style="Detail_Header" x="453" y="30" width="75" height="20" uuid="f696129a-023d-40be-962c-5692865fabcc">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<textElement textAlignment="Right" markup="none">
					<font size="8" pdfFontName="Helvetica-Bold"/>
				</textElement>
				<textFieldExpression><![CDATA[$P{inpLabel1}]]></textFieldExpression>
			</textField>
			<staticText>
				<reportElement key="staticText-25" style="Detail_Header" x="410" y="30" width="43" height="20" uuid="e49768c3-1035-4b57-9ff2-c2370194685d">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitwidth" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
				</reportElement>
				<box leftPadding="5" rightPadding="2">
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#FFFFFF"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#FFFFFF"/>
				</box>
				<textElement textAlignment="Center">
					<font size="8" pdfFontName="Helvetica-Bold"/>
				</textElement>
				<text><![CDATA[Dias]]></text>
			</staticText>
			<staticText>
				<reportElement key="staticText-25" style="Detail_Header" x="0" y="30" width="50" height="20" uuid="95dba1e2-4c15-4cc0-b080-b52a398e12cb">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitwidth" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
				</reportElement>
				<box leftPadding="5" rightPadding="2">
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#FFFFFF"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#FFFFFF"/>
				</box>
				<textElement textAlignment="Left">
					<font size="8" pdfFontName="Helvetica-Bold"/>
				</textElement>
				<text><![CDATA[RUC]]></text>
			</staticText>
			<line>
				<reportElement x="453" y="0" width="1" height="30" uuid="c02f6ddf-0c97-4b13-b079-5b5e65605d4f">
					<property name="local_mesure_unitheight" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.height" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement x="828" y="0" width="1" height="30" uuid="ffad5a8d-08a7-4611-b145-df152f37a4a7">
					<property name="local_mesure_unitheight" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.height" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement x="1203" y="0" width="1" height="30" uuid="f8296604-2aaf-461d-bc56-148321a28d6f">
					<property name="local_mesure_unitheight" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.height" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement x="453" y="0" width="750" height="1" uuid="a4ee006a-fa40-4483-9f91-d2f34c55c529">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitwidth" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
				</reportElement>
			</line>
			<staticText>
				<reportElement x="453" y="3" width="375" height="20" uuid="5b6c7759-b40d-4a08-b847-4f329e14b38a">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitheight" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.height" value="px"/>
					<property name="local_mesure_unitwidth" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<textElement textAlignment="Center">
					<font size="10" isBold="true"/>
				</textElement>
				<text><![CDATA[POR VENCER]]></text>
			</staticText>
			<staticText>
				<reportElement x="828" y="3" width="375" height="20" uuid="8db09612-c9e7-41d9-9439-c9c15baee4bc">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitheight" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.height" value="px"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<textElement textAlignment="Center">
					<font isBold="true"/>
				</textElement>
				<text><![CDATA[VENCIDOS]]></text>
			</staticText>
```

Este fragmento de código estructura y rotula el reporte de manera clara, organizando los encabezados para identificar correctamente las columnas correspondientes a cada rango de días (0–30, 31–60, etc.). Adicionalmente, muestra el RUC del cliente para evitar errores de identificación y define encabezados diferenciados que separan visualmente las obligaciones por vencer de aquellas ya vencidas.



También se observó lo siguiente:

```
<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Line" x="1053" y="0" width="75" height="16" forecolor="#000000" uuid="a87a1b93-e880-4ffe-a8a6-38052efe3e99">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="8" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$F{AMOUNT4} != null ? $P{AMOUNTFORMAT}.format($F{AMOUNT4}) : null]]></textFieldExpression>
			</textField>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Line" x="978" y="0" width="75" height="16" forecolor="#000000" uuid="3b70cb66-8c80-46f3-b579-badd9e263515">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="8" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$F{AMOUNT3} != null ? $P{AMOUNTFORMAT}.format($F{AMOUNT3}) : null]]></textFieldExpression>
			</textField>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Line" x="903" y="0" width="75" height="16" forecolor="#000000" uuid="672a5833-4be5-4fba-9239-7c53321b518f">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="8" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$F{AMOUNT2} != null ? $P{AMOUNTFORMAT}.format($F{AMOUNT2}) : null]]></textFieldExpression>
			</textField>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Line" x="828" y="0" width="75" height="16" forecolor="#000000" uuid="152fe270-ec2a-484d-9430-e27213879033">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="8" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$F{AMOUNT1} != null ? $P{AMOUNTFORMAT}.format($F{AMOUNT1}) : null]]></textFieldExpression>
			</textField>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Line" x="410" y="0" width="43" height="16" forecolor="#000000" uuid="29c69baf-db87-432e-9068-32316b671c52">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Center" verticalAlignment="Middle">
					<font size="8" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$F{dias}]]></textFieldExpression>
			</textField>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Line" x="296" y="0" width="114" height="16" forecolor="#000000" uuid="a6846b45-2b2d-4655-ba2d-dd41cf8fff5b">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.width" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Center" verticalAlignment="Middle">
					<font size="8" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$F{dueDate}]]></textFieldExpression>
			</textField>
```

Este componente es responsable de registrar la información efectiva presentada en el reporte, incluyendo los valores adeudados en cada rango de tiempo, la fecha de vencimiento de cada factura y el cálculo automático de los días de atraso correspondientes al cliente.

Y para finalizar este análisis,

```
	</reportElement>
			</line>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Header" x="1053" y="48" width="76" height="16" forecolor="#000000" backcolor="#FFFFFF" uuid="7203494e-be49-4605-9573-0bc1d7403098">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="8" isBold="false" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$V{SUMAMT4TOTAL} != null ? $P{AMOUNTFORMAT}.format($V{SUMAMT4TOTAL}) : $P{AMOUNTFORMAT}.format(new BigDecimal("0"))]]></textFieldExpression>
			</textField>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Header" x="978" y="48" width="74" height="16" forecolor="#000000" backcolor="#FFFFFF" uuid="b2012274-7ad6-4a09-92e2-d06a76b94272">
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="8" isBold="false" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$V{SUMAMT3TOTAL} != null ? $P{AMOUNTFORMAT}.format($V{SUMAMT3TOTAL}) : $P{AMOUNTFORMAT}.format(new BigDecimal("0"))]]></textFieldExpression>
			</textField>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Header" x="903" y="48" width="76" height="16" forecolor="#000000" backcolor="#FFFFFF" uuid="4950f762-4343-44db-9518-db31b0890270">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="8" isBold="false" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$V{SUMAMT2TOTAL} != null ? $P{AMOUNTFORMAT}.format($V{SUMAMT2TOTAL}) : $P{AMOUNTFORMAT}.format(new BigDecimal("0"))]]></textFieldExpression>
			</textField>
			<textField isBlankWhenNull="true" hyperlinkType="Reference">
				<reportElement key="textField-6" style="Detail_Header" x="828" y="48" width="74" height="16" forecolor="#000000" backcolor="#FFFFFF" uuid="25823e03-a010-4ecf-8fd3-dd925fd77f8f">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box leftPadding="2"/>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="8" isBold="false" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[$V{SUMAMT1TOTAL} != null ? $P{AMOUNTFORMAT}.format($V{SUMAMT1TOTAL}) : $P{AMOUNTFORMAT}.format(new BigDecimal("0"))]]></textFieldExpression>
			</textField>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="987" y="45" width="71" height="1" uuid="61d64a34-27e1-4b30-9bed-fad50c582aed">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="837" y="45" width="71" height="1" uuid="f9e93a81-82f0-46ee-a766-339b7e992eaa">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="912" y="45" width="71" height="1" uuid="9dd96dd5-2898-47a1-a6cb-19da616847fa">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="1062" y="45" width="71" height="1" uuid="9e2297fb-fe21-41cd-a6a9-b11f7b86b22a">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="837" y="66" width="71" height="1" uuid="a6018cd9-c77d-4f65-87b6-7b7b4cb2e2f3">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="912" y="66" width="71" height="1" uuid="9eaaf595-7eea-40ad-b5c6-ffecb6c28843">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="987" y="66" width="71" height="1" uuid="2e1bd6a8-7eaa-43f4-a312-1c90c3dfb421">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="1062" y="66" width="71" height="1" uuid="fe597819-a0d0-4d5c-a38f-2eaa4f8821a6">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="837" y="68" width="71" height="1" uuid="924ddc24-c3f5-4ecc-8217-91ca66828156">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="912" y="68" width="71" height="1" uuid="569f28d2-5f98-4296-9cef-598a1449e3a8">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="987" y="68" width="71" height="1" uuid="da131bdf-77f4-4d88-8417-93ea25020a89">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
				</reportElement>
			</line>
			<line>
				<reportElement key="line-33" style="Report_Footer" x="1062" y="68" width="71" height="1" uuid="db8fba2b-ba90-4a0f-995e-7826b09e3417">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
					<property name="local_mesure_unity" value="pixel"/>
					<property name="local_mesure_unitx" value="pixel"/>
				</reportElement>
```

Este codigo corresponde a la fila de **Gran Total**, la cual consolida todas las deudas de la totalidad de clientes, realizando la suma por columnas y aplicando los indicadores visuales de cierre contable al final de la última página del reporte. Este elemento es indispensable para identificar el monto total pendiente de cobro o pago

#### Es óptimo integrarlo

Sí, es altamente recomendable, aunque con una consideración técnica. Se debe conservar el diseño de la versión 18, ya que es más completo y se ajusta mejor a la realidad operativa del negocio en Ecuador. No obstante, la versión 24 incorpora la propiedad `textAdjust="StretchHeight"`, por lo que es necesario adaptar el archivo de la versión 18 para que utilice esta propiedad en lugar de `isStretchWithOverflow`, evitando incompatibilidades con el motor actualizado de JasperReports. Adicionalmente, se debe verificar y ajustar los demás archivos asociados a este reporte para asegurar su correcto funcionamiento



### Ticket 10095 - BUG - NOMBRES EN REPORTE DE ANTIGUEDAD DE COBROS - PAGOS - GLPI 5107

Este reporte proporciona al área financiera un resumen ejecutivo de los saldos adeudados, indicando quién mantiene la deuda y el estado en el que se encuentra, sin detallar factura por factura, ya que dicho nivel de información es cubierto por el reporte anterior.

Se identificaron las siguientes diferencias:

```
<line>
					<reportElement key="line-33" style="Report_Footer" x="260" y="0" width="48" height="1" uuid="dab89c4e-70b6-417e-b937-ad4600b1eb97">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="310" y="0" width="48" height="1" uuid="ee539e8a-4559-4e95-b2c6-f72736a148c2">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="360" y="0" width="48" height="1" uuid="3e5dfaf0-9b30-4384-af6a-dd22c1bd2363">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="410" y="0" width="48" height="1" uuid="b6e5e58a-0176-4218-9218-63182a4be27f">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" x="460" y="0" width="48" height="1" uuid="a7cd2a15-9223-4eb5-a56c-82c7bd1ecacb">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<textField isStretchWithOverflow="true" pattern="##0.00" isBlankWhenNull="false" hyperlinkType="Reference">
					<reportElement key="textField-2" mode="Opaque" x="205" y="2" width="50" height="14" forecolor="#000000" backcolor="#FFFFFF" uuid="4f54a360-55bc-40c0-9ab9-d163dcaf1434">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
					<box>
						<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
						<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
						<bottomPen lineWidth="0.0" lineColor="#000000"/>
						<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					</box>
					<textElement textAlignment="Right" verticalAlignment="Middle">
						<font size="6" isUnderline="false"/>
					</textElement>
					<textFieldExpression><![CDATA[($V{SUM_ONE_11}!=null)?$P{AMOUNTFORMAT}.format($V{SUM_ONE_11}):new String(" ")]]></textFieldExpression>
				</textField>
				<textField isStretchWithOverflow="true" pattern="##0.00" isBlankWhenNull="false" hyperlinkType="Reference">
					<reportElement key="textField-2" mode="Opaque" x="255" y="2" width="50" height="14" forecolor="#000000" backcolor="#FFFFFF" uuid="b2b2a8ab-ede2-48cb-b050-9eaa49a60dde">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
					<box>
						<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
						<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
						<bottomPen lineWidth="0.0" lineColor="#000000"/>
						<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					</box>
					<textElement textAlignment="Right" verticalAlignment="Middle">
						<font size="6" isUnderline="false"/>
					</textElement>
					<textFieldExpression><![CDATA[($V{SUM_TWO_12}!=null)?$P{AMOUNTFORMAT}.format($V{SUM_TWO_12}):new String(" ")]]></textFieldExpression>
				</textField>
				<textField isStretchWithOverflow="true" pattern="##0.00" isBlankWhenNull="false" hyperlinkType="Reference">
					<reportElement key="textField-2" mode="Opaque" x="305" y="2" width="50" height="14" forecolor="#000000" backcolor="#FFFFFF" uuid="b557966b-06bd-4ab2-bdfd-73255c3ea5cc">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
					<box>
						<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
						<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
						<bottomPen lineWidth="0.0" lineColor="#000000"/>
						<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					</box>
					<textElement textAlignment="Right" verticalAlignment="Middle">
						<font size="6" isUnderline="false"/>
					</textElement>
					<textFieldExpression><![CDATA[($V{SUM_THREE_13}!=null)?$P{AMOUNTFORMAT}.format($V{SUM_THREE_13}):new String(" ")]]></textFieldExpression>
				</textField>
				<textField isStretchWithOverflow="true" pattern="##0.00" isBlankWhenNull="false" hyperlinkType="Reference">
					<reportElement key="textField-2" mode="Opaque" x="355" y="2" width="50" height="14" forecolor="#000000" backcolor="#FFFFFF" uuid="2c560668-b425-4524-8215-33857145ae52">
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
						<property name="local_mesure_unitx" value="pixel"/>
					</reportElement>
					<box>
						<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
						<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
						<bottomPen lineWidth="0.0" lineColor="#000000"/>
						<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					</box>
					<textElement textAlignment="Right" verticalAlignment="Middle">
						<font size="6" isUnderline="false"/>
					</textElement>
					<textFieldExpression><![CDATA[($V{SUM_FOUR_14}!=null)?$P{AMOUNTFORMAT}.format($V{SUM_FOUR_14}):new String(" ")]]></textFieldExpression>
				</textField>
				<line>
					<reportElement key="line-33" style="Report_Footer" positionType="FixRelativeToBottom" x="210" y="17" width="48" height="1" uuid="1699f38b-17d1-45d3-a4ce-269d75adec4b">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" positionType="FixRelativeToBottom" x="260" y="17" width="48" height="1" uuid="0f9c0d7f-8de4-4650-a1cb-a0c0824cf902">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" positionType="FixRelativeToBottom" x="310" y="17" width="48" height="1" uuid="75a05724-d57d-4faa-8352-91b18a31776e">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" positionType="FixRelativeToBottom" x="360" y="17" width="48" height="1" uuid="2091f6ab-8046-4ed6-a6ed-0f0757717ec4">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" positionType="FixRelativeToBottom" x="210" y="19" width="48" height="1" uuid="4813f320-c870-48b9-a3e9-924965c1f8f6">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" positionType="FixRelativeToBottom" x="260" y="19" width="48" height="1" uuid="3147fabc-a59f-4947-a8fe-e349cea97283">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" positionType="FixRelativeToBottom" x="310" y="19" width="48" height="1" uuid="46f1d60f-4aa4-4431-8943-fbf03d591caa">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
				<line>
					<reportElement key="line-33" style="Report_Footer" positionType="FixRelativeToBottom" x="360" y="19" width="48" height="1" uuid="42ad91d6-0b9e-440d-aef7-cb0fadbc4f4c">
						<property name="local_mesure_unitx" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.x" value="px"/>
						<property name="local_mesure_unity" value="pixel"/>
						<property name="com.jaspersoft.studio.unit.y" value="px"/>
					</reportElement>
				</line>
```

Esta sección del código corresponde al pie de página del grupo (Group Footer) y tiene como función generar los subtotales por cada cliente dentro del Reporte de Antigüedad de Cartera

También se observó lo siguiente:

```
<textField isStretchWithOverflow="true" pattern="##0.00" isBlankWhenNull="false" hyperlinkType="Reference">
				<reportElement key="textField" stretchType="RelativeToTallestObject" x="205" y="0" width="50" height="16" isPrintWhenDetailOverflows="true" forecolor="#000000" uuid="3b2133c9-f132-474c-8d91-8e4fbe68d6d7">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box>
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
				</box>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="6" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[($F{amount11}!=null)?$P{AMOUNTFORMAT}.format($F{amount11}):new String(" ")]]></textFieldExpression>
			</textField>
			<textField isStretchWithOverflow="true" pattern="##0.00" isBlankWhenNull="false" hyperlinkType="Reference">
				<reportElement key="textField" stretchType="RelativeToTallestObject" x="255" y="0" width="50" height="16" isPrintWhenDetailOverflows="true" forecolor="#000000" uuid="9fff0982-4793-444e-a352-5f64b78f11b2">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box>
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
				</box>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="6" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[($F{amount12}!=null)?$P{AMOUNTFORMAT}.format($F{amount12}):new String(" ")]]></textFieldExpression>
			</textField>
			<textField isStretchWithOverflow="true" pattern="##0.00" isBlankWhenNull="false" hyperlinkType="Reference">
				<reportElement key="textField" stretchType="RelativeToTallestObject" x="305" y="0" width="50" height="16" isPrintWhenDetailOverflows="true" forecolor="#000000" uuid="c7623b62-b204-4119-91e9-91dc429c2c43">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box>
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
				</box>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="6" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[($F{amount13}!=null)?$P{AMOUNTFORMAT}.format($F{amount13}):new String(" ")]]></textFieldExpression>
			</textField>
			<textField isStretchWithOverflow="true" pattern="##0.00" isBlankWhenNull="false" hyperlinkType="Reference">
				<reportElement key="textField" stretchType="RelativeToTallestObject" x="355" y="0" width="50" height="16" isPrintWhenDetailOverflows="true" forecolor="#000000" uuid="4681006a-805d-48df-a599-5e96e71152a2">
					<property name="com.jaspersoft.studio.unit.x" value="px"/>
					<property name="local_mesure_unitx" value="pixel"/>
					<property name="com.jaspersoft.studio.unit.y" value="px"/>
				</reportElement>
				<box>
					<topPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<leftPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
					<bottomPen lineWidth="0.0" lineColor="#000000"/>
					<rightPen lineWidth="0.0" lineStyle="Solid" lineColor="#000000"/>
				</box>
				<textElement textAlignment="Right" verticalAlignment="Middle">
					<font size="6" isUnderline="false"/>
				</textElement>
				<textFieldExpression><![CDATA[($F{amount14}!=null)?$P{AMOUNTFORMAT}.format($F{amount14}):new String(" ")]]></textFieldExpression>
			</textField>
```

Este código es el encargado de asignar los valores monetarios a las columnas correspondientes a los distintos rangos de tiempo del reporte. Su correcta implementación es fundamental para que el área financiera pueda identificar con precisión el monto adeudado por cada cliente en los periodos de mora más antiguos.\
En consecuencia, se identificaron las siguientes diferencias:

* La versión 18 permite visualizar un desglose más detallado de los rangos de vencimiento, incorporando un mayor número de columnas y facilitando la lectura al separar de forma visual las obligaciones vigentes de aquellas que se encuentran en estado de mora.
* La versión 24 presenta una tabla simple que cumple con el estándar base del sistema, pero resulta limitada para las necesidades operativas del área de cobranzas.



### Ticket 3428 - "3428 DESARROLLO - Corrección formato salida Antiguedad de cartera"

&#x20;En la versión 18, Sidesoft redefine las rutas de los archivos de exportación a Excel para utilizar versiones personalizadas que soportan un mayor número de columnas y etiquetas en idioma español.

Ubicación de las rutas:

```
private static final String AGING_SCHEDULE_XLS = BASE_DESIGN
      + "/ec/com/sidesoft/custom/reports/ad_reports/estandar/finances/AgingScheduleXLS.jrxml";

  private static final String AGING_SCHEDULE_DETAIL_XLS = BASE_DESIGN
      + "/ec/com/sidesoft/custom/reports/ad_reports/estandar/finances/AgingScheduleDetailXLS.jrxml";
```

Otro cambio corresponde a la personalización de los nombres, donde se definen denominaciones específicas para los archivos y encabezados, permitiendo que los reportes exportados utilicen nombres más descriptivos y acordes a la terminología del negocio.

```
parameters.put("inpLabel1", "1-" + strColumn1);
        parameters.put("inpLabel2", (Integer.valueOf(strColumn1) + 1) + "-" + strColumn2);
        parameters.put("inpLabel3", (Integer.valueOf(strColumn2) + 1) + "-" + strColumn3);
        parameters.put("inpLabel4", (Integer.valueOf(strColumn3) + 1) + "-" + strColumn4);
        parameters.put("inpLabel5", "Sobre " + strColumn4); // <-- Aquí dice "Sobre" en lugar de "Over"
```

Finalmente, se incorpora la definición de títulos dinámicos, los cuales se generan en función de los filtros y parámetros seleccionados, permitiendo que cada reporte refleje de forma precisa el contexto de la información presentada.

```
if (RECEIVABLES.equals(recOrPay)) {
          parameters.put("title", Utility.messageBD(conn, "AGING_ORASD", vars.getLanguage()));
          parameters.put("tabID", "263");
          parameters.put("tabTitle", Utility.messageBD(conn, "AGING_ORASD", vars.getLanguage()));
        } else {
          parameters.put("title", Utility.messageBD(conn, "AGING_PASD", vars.getLanguage()));
          parameters.put("tabID", "290");
          parameters.put("tabTitle", Utility.messageBD(conn, "AGING_PASD", vars.getLanguage()));
        }
```

Todo lo anterior evita que el reporte en formato Excel presente errores de formato, asegurando el uso correcto de los diseños ampliados definidos por Sidesoft.\
¿Es óptimo optimizarlo? Sí. Se recomienda integrar las rutas de archivos personalizadas de Sidesoft y las etiquetas en español provenientes de la versión 18 dentro del código modernizado de la versión 24, garantizando compatibilidad, legibilidad y correcto funcionamiento.



_**En conclusión, el análisis demuestra que la versión 24 debe utilizarse como base, ya que cuenta con una arquitectura más actual, mayor compatibilidad con los motores modernos y una mejor estabilidad general. No obstante, la versión 18 incorpora varias personalizaciones y reglas de negocio que han demostrado ser útiles en la operación diaria, especialmente en los procesos contables, logísticos y de cobranzas, así como en la generación de reportes adaptados a la realidad del negocio en Ecuador. Por este motivo, la mejor alternativa es integrar de forma selectiva aquellas funcionalidades de la versión 18 que realmente aportan valor, adaptándolas al código y estándares de la versión 24, de manera que se mantenga la facilidad de uso, la trazabilidad y la identidad visual del sistema, sin afectar la seguridad, el rendimiento ni la estabilidad del entorno AgilERP24Q4.1.**_





