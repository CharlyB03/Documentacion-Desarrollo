# Documentacion Modulo org.openbravo.advpaymentmngt



> org.openbravo.advpaymentmngt

Cambios que se realizaron en este modulo, especificamente en  estos 2 modulos

> ec.com.sidesoft.custom.advpaymentmngt.actionHandler
>
> org.openbravo.advpaymentmngt.actionHandler



**Se agrego esta linea dentro de la V18**

```java
transaction.setLineNo(TransactionsDao.getTransactionMaxLineNo(account) + 10);
```

Lo que hace es colocar a la transaccion un numero de linea, luego la linea mayor es decir la ultima y la suma +10, digamos si le coloco a la transaccion 40 el nuveo numero de quedaria 50

**Ademas se agrego el siguiente metodo dentro de la V18**

```java
public static FIN_FinaccTransaction createFinAccTransaction(FIN_Payment payment,
      String strDeposit) {
    final FIN_FinaccTransaction newTransaction = OBProvider.getInstance()
        .get(FIN_FinaccTransaction.class);
    OBContext.setAdminMode();
    try {
      newTransaction.setId(payment.getId());
      newTransaction.setNewOBObject(true);
      newTransaction.setFinPayment(payment);
      newTransaction.setOrganization(payment.getOrganization());
      newTransaction.setAccount(payment.getAccount());
      newTransaction.setDateAcct(payment.getPaymentDate());
      newTransaction.setTransactionDate(payment.getPaymentDate());
      newTransaction.setActivity(payment.getActivity());
      newTransaction.setProject(payment.getProject());
      newTransaction.setCostCenter(payment.getCostCenter());
      newTransaction.setStDimension(payment.getStDimension());
      newTransaction.setNdDimension(payment.getNdDimension());
      newTransaction.setCurrency(payment.getAccount().getCurrency());
      String desc = "";
      if (payment.getDescription() != null && !payment.getDescription().isEmpty()) {
        desc = payment.getDescription().replace("\n", ". ").substring(0,
            payment.getDescription().length() > 254 ? 254 : payment.getDescription().length());
      }
      newTransaction.setDescription(desc);
      newTransaction.setClient(payment.getClient());
      newTransaction.setLineNo(getTransactionMaxLineNo(payment.getAccount()) + 10);

      BigDecimal depositAmt = FIN_Utility.getDepositAmount(
          payment.getDocumentType().getDocumentCategory().equals("ARR"),
          payment.getFinancialTransactionAmount());
      BigDecimal paymentAmt = FIN_Utility.getPaymentAmount(
          payment.getDocumentType().getDocumentCategory().equals("ARR"),
          payment.getFinancialTransactionAmount());

      newTransaction.setDepositAmount(depositAmt);
      newTransaction.setPaymentAmount(paymentAmt);
      newTransaction.setStatus(
          newTransaction.getDepositAmount().compareTo(newTransaction.getPaymentAmount()) > 0 ? "RPR"
              : "PPM");
      if (!newTransaction.getCurrency().equals(payment.getCurrency())) {
        newTransaction.setForeignCurrency(payment.getCurrency());
        newTransaction.setForeignConversionRate(payment.getFinancialTransactionConvertRate());
        newTransaction.setForeignAmount(payment.getAmount());
      }
      payment.getFINFinaccTransactionList().add(newTransaction);

      if (!strDeposit.equals("ND") && payment.isReceipt()) {

        FIN_Payment depositPayment = payment;
        depositPayment.setEcscapDeposit(strDeposit);
        OBDal.getInstance().save(depositPayment);
      }

      OBDal.getInstance().save(newTransaction);
      OBDal.getInstance().flush();
    } finally {
      OBContext.restorePreviousMode();
    }
    return newTransaction;
  }

  @SuppressWarnings("deprecation")
  public static Long getTransactionMaxLineNo(FIN_FinancialAccount financialAccount) {
    Query query = OBDal.getInstance().getSession().createQuery(
        "select max(f.lineNo) as maxLineno from FIN_Finacc_Transaction as f where account.id=?");
    query.setString(0, financialAccount.getId());
    for (Object obj : query.list()) {
      if (obj != null) {
        return (Long) obj;
      }
    }
    return 0l;
  }
```

Este método se encarga de procesar la información del pago, construyendo un registro de transacción bancaria ordenado por número de línea, con los montos y la moneda correctamente definidos, y vinculándolo de forma directa al pago correspondiente.





### Comparacion con la V24

Codigo nuevo encontrado en la V24

```java
@Inject
  @Any
  private Instance<AddMultiplePaymentsProcessAfterProcessHook> afterHooks;

```

Ademas se encontro  dentro de la V24

```java
List<AddMultiplePaymentsProcessAfterProcessHook> hooksPriority = new ArrayList<AddMultiplePaymentsProcessAfterProcessHook>();
  for (AddMultiplePaymentsProcessAfterProcessHook hook : afterHooks) {
    hooksPriority.add(hook);
  }
  Collections.sort(hooksPriority, new Comparator<AddMultiplePaymentsProcessAfterProcessHook>() {
    @Override
    public int compare(AddMultiplePaymentsProcessAfterProcessHook o1,
        AddMultiplePaymentsProcessAfterProcessHook o2) {
      return (int) Math.signum(o2.getPriority() - o1.getPriority());
    }
  });
  for (AddMultiplePaymentsProcessAfterProcessHook hook : hooksPriority) {
    selectedPaymentsLength = selectedPaymentsLength + hook.executeHook(jsonData);
  }
```

En conjunto lo que hace es buscar todas las funciones adicionales creadas en cualquier parte del proyecto ,luego las ordena por importancia (prioridad) para que lo más urgente se haga primero y ejecuta cada tarea una por una de forma secuencia



## Es optimo implementar lo de la v18 a la v24

Desde este lado, y para evitar problemas de duplicidad en las transacciones, se observo esta nueva customización realizada en la V18. Si bien en la V24 ya existe el TransactionDao, en este caso se detectó que había un alto riesgo de que se repita el número de línea.

Con este cambio se corrige ese problema, ya que el nuevo método maneja la numeración correctamente, sumando +10 y evitando así cualquier conflicto o duplicidad.





### Análisis de la clase AddPaymentActionHandler.java entre Estándar V18 y ÁgilERP V18

En este caso, se realizará la comparación entre una clase customizada y una versión limpia, sin modificaciones

> Modulo: ec.com.sidesoft.creditcard.reconciliation
>
> Clase: SSCCR\_AddPaymentActionHandler.java
>
>
>
> Modulo : org.openbravo.advpaymentmngt
>
> Clase: SSCCR\_AddPaymentActionHandler.java

Lo que logramos observar en esta implementación V18 es lo siguiente:

```java
   if (jsonparams.get("reference_no") == JSONObject.NULL) {
        String strfin_paymentmethod_id = jsonparams.getString("fin_paymentmethod_id");

        FIN_PaymentMethod paymentMethod = OBDal.getInstance().get(FIN_PaymentMethod.class,
            strfin_paymentmethod_id);
        if (paymentMethod != null && paymentMethod.getSccaTypePayment() != null
            && paymentMethod.getSccaTypePayment().equals("CA")) {
          OBDal.getInstance().rollbackAndClose();
          JSONObject errorMessage = new JSONObject();
          errorMessage.put("severity", "error");
          errorMessage.put("text", "El número de referencia no puede estar vacío.");
          jsonResponse.put("retryExecution", openedFromMenu);
          jsonResponse.put("message", errorMessage);
          return jsonResponse;
        }
      }
```

Esto nos permite validar un pago. Si el JSON recibe el campo `reference_no` como nulo, se carga el método de pago (`FIN_PaymentMethod`) y, en caso de que dicho método sea de tipo **“CA”**, se aborta la operación (rollback). Posteriormente, se construye una respuesta en formato JSON con severidad **error**, devolviendo el mensaje: “El número de referencia no puede estar vac**ío”.**

Además, encontramos que en la V18:

```java
   removeNotSelectedPaymentDetails(payment, pdToRemove);

      if (strAction.equals("PRP") || strAction.equals("PPP") || strAction.equals("PRD")
          || strAction.equals("PPW")) {

        OBError message = processPayment(payment, strAction, strDifferenceAction, differenceAmount,
            exchangeRate, jsonparams, comingFrom);

        // SE VALIDAN LOS PARAMETROS CUANDO ES METODO DE PAGO TARJETA
        if (jsonparams.has("ssccr_types_of_credit_id")
            && jsonparams.has("Ssccr_Processor_Banck_ID")) {

          String strSsccrTypesOfCreditId = jsonparams.getString("ssccr_types_of_credit_id");
          String strSsccrProcessorBanckId = jsonparams.getString("Ssccr_Processor_Banck_ID");

          if (!strSsccrTypesOfCreditId.equals("null") && !strSsccrProcessorBanckId.equals("null")) {

            if (strAction.equals("PRD")) {

              String strFinancialAccountId = jsonparams.getString("fin_financial_account_id");
              FIN_FinancialAccount finAccount = OBDal.getInstance().get(FIN_FinancialAccount.class,
                  strFinancialAccountId);

              final OBCriteria<FIN_FinaccTransaction> obc = OBDal.getInstance()
                  .createCriteria(FIN_FinaccTransaction.class);
              obc.add(Restrictions.eq(FIN_FinaccTransaction.PROPERTY_FINPAYMENT, payment));
              obc.add(Restrictions.eq(FIN_FinaccTransaction.PROPERTY_ACCOUNT, finAccount));

              String strbanktransferId = jsonparams.getString("ssfi_banktransfer_id");
              ssfiBanktransfer objSsfiBanktransfer = OBDal.getInstance().get(ssfiBanktransfer.class,
                  strbanktransferId);

              SsccrTypesOfCredit objSsccrTypesOfCredit = OBDal.getInstance()
                  .get(SsccrTypesOfCredit.class, strSsccrTypesOfCreditId);

              SsccrProcessorBanck objSsccrProcessorBanck = OBDal.getInstance()
                  .get(SsccrProcessorBanck.class, strSsccrProcessorBanckId);

              String strC_Order_ID = jsonparams.getString("c_order_id");

              String strSsccrCardsTypesId = jsonparams.getString("Ssccr_Cards_Types_ID");
              SsccrCardsTypes objSsccrCardsTypes = OBDal.getInstance().get(SsccrCardsTypes.class,
                  strSsccrCardsTypesId);

              String strInvoiceNo = "";
              String strSalesOrderNo = "";
              Order objOrder = null;
              Invoice objInvoice = null;
              JSONObject orderInvoiceGrid = jsonparams.getJSONObject("order_invoice");
              JSONArray selectedPSDs = orderInvoiceGrid.getJSONArray("_selection");
              final OBCriteria<Invoice> objInvoiceLst = OBDal.getInstance()
                  .createCriteria(Invoice.class);

              if (strC_Order_ID == null || strC_Order_ID.equals("")
                  || strC_Order_ID.equals("null")) {
                for (int i = 0; i < selectedPSDs.length(); i++) {

                  JSONObject psdRow = selectedPSDs.getJSONObject(i);
                  strInvoiceNo = psdRow.getString("invoiceNo");
                  strSalesOrderNo = psdRow.getString("salesOrderNo");

                  if (strSalesOrderNo == null || strSalesOrderNo.equals("")) {
                    objInvoiceLst.add(Restrictions.eq(Invoice.PROPERTY_DOCUMENTNO, strInvoiceNo));
                  } else if (strInvoiceNo != null || !strInvoiceNo.equals("")) {
                    final OBCriteria<Order> objOrderLst = OBDal.getInstance()
                        .createCriteria(Order.class);
                    objOrderLst.add(Restrictions.eq(Order.PROPERTY_DOCUMENTNO, strSalesOrderNo));
                    objOrder = objOrderLst.list().get(0);
                    objInvoiceLst.add(Restrictions.eq(Invoice.PROPERTY_SALESORDER, objOrder));

                  }
                }
              } else {
                objOrder = OBDal.getInstance().get(Order.class, strC_Order_ID);
                objInvoiceLst.add(Restrictions.eq(Invoice.PROPERTY_SALESORDER, objOrder));
              }

              if (objInvoiceLst != null && !objInvoiceLst.equals("")) {
                if (selectedPSDs.length() > 0) {
                  for (Invoice invoiceobj : objInvoiceLst.list()) {
                    objInvoice = invoiceobj;
                    objOrder = invoiceobj.getSalesOrder();
                  }
                }
              }

              Calendar c = Calendar.getInstance();
              c.setTime(paymentDate);

              // Busqueda de Configuración conciliación de tarjetas
              if (objSsccrCardsTypes != null) {
                String value = new SimpleDateFormat("yyyy-MM-dd").format(paymentDate);
                ConnectionProvider conn = null;
                try {
                  conn = new DalConnectionProvider(true);
                  String strPaymentTermId = SSCCRPaymentTermData.paymentterm(conn,
                      objSsccrCardsTypes.getId(), objSsccrTypesOfCredit.getId(), value);
                  if (strPaymentTermId != null && !strPaymentTermId.equals("")) {
                    PaymentTerm objPaymentTerm = OBDal.getInstance().get(PaymentTerm.class,
                        strPaymentTermId);
                    c.add(Calendar.DAY_OF_YEAR,
                        objPaymentTerm.getOverduePaymentDaysRule().intValue());
                  } else {
                    OBDal.getInstance().rollbackAndClose();
                    JSONObject errorMessage = new JSONObject();
                    errorMessage.put("severity", "error");
                    errorMessage.put("text",
                        "No existe configuración de condición de pago para los parámetros dados.");
                    jsonResponse.put("retryExecution", openedFromMenu);
                    jsonResponse.put("message", errorMessage);
                    return jsonResponse;
                  }
                } catch (Exception ex) {
                  ex.printStackTrace();
                } finally {
                  conn.destroy();
                }
              }
              String lot = jsonparams.getString("Lote");

              for (FIN_FinaccTransaction financialAccount : obc.list()) {
                financialAccount.setSsccrSsfiBanktransfer(objSsfiBanktransfer);
                financialAccount.setSsccrTypesOfCredit(objSsccrTypesOfCredit);
                financialAccount.setSsccrProcessorBanck(objSsccrProcessorBanck);
                financialAccount.setSsccrCardsTypes(objSsccrCardsTypes);
                financialAccount.setSsccrCInvoice(objInvoice);
                financialAccount.setSsccrCOrder(objOrder);
                financialAccount.setSsccrFinPayment(payment);
                financialAccount.setSsccrLot(lot);
                if (objSsccrProcessorBanck != null) {
                  financialAccount.setSsccrExpirationDate(c.getTime());
                }
                if (jsonparams.get("reference_no") != JSONObject.NULL) {
                  financialAccount.setSsccrRecapno(jsonparams.getString("reference_no"));
                }
                try {
                  OBDal.getInstance().save(financialAccount);
                  OBDal.getInstance().flush();
                } finally {
                }
              }
            }

          }

        }
```

Esta implementación nos permite procesar un pago y, en caso de que se trate de un pago con tarjeta y la acción sea PRD, se actualiza la transacción bancaria asociada (`FIN_FinaccTransaction`) con la información de conciliación correspondiente, como el tipo de crédito, procesador o banco, tarjeta, lote, factura u orden y la fecha de expiración. Además, se guarda el número de referencia cuando este existe.



### Comparativa entre la versión Estándar V18 y ÁgilERP V24

Clases comparadas entre versiones

Correspondientes a la versión Estándar V18

> &#x20;SSCCR\_AddPaymentActionHandler.java

Con la versión ÁgilERP V24

> AddPaymentActionHandler.java

En la V24 se repiten los mismos cambios que en el caso anterior.

Lo que logramos observar en esta implementación en la V18 es lo siguiente:

```java
   if (jsonparams.get("reference_no") == JSONObject.NULL) {
        String strfin_paymentmethod_id = jsonparams.getString("fin_paymentmethod_id");

        FIN_PaymentMethod paymentMethod = OBDal.getInstance().get(FIN_PaymentMethod.class,
            strfin_paymentmethod_id);
        if (paymentMethod != null && paymentMethod.getSccaTypePayment() != null
            && paymentMethod.getSccaTypePayment().equals("CA")) {
          OBDal.getInstance().rollbackAndClose();
          JSONObject errorMessage = new JSONObject();
          errorMessage.put("severity", "error");
          errorMessage.put("text", "El número de referencia no puede estar vacío.");
          jsonResponse.put("retryExecution", openedFromMenu);
          jsonResponse.put("message", errorMessage);
          return jsonResponse;
        }
      }
```

Esta validación permite controlar el pago. En caso de que el JSON no incluya el campo `reference_no`, se carga el método de pago (`FIN_PaymentMethod`) y, si este es de tipo **CA**, se realiza un rollback y se devuelve una respuesta JSON con severidad error.

Además, encontramos en la V18 lo siguiente:

```java
   removeNotSelectedPaymentDetails(payment, pdToRemove);

      if (strAction.equals("PRP") || strAction.equals("PPP") || strAction.equals("PRD")
          || strAction.equals("PPW")) {

        OBError message = processPayment(payment, strAction, strDifferenceAction, differenceAmount,
            exchangeRate, jsonparams, comingFrom);

        // SE VALIDAN LOS PARAMETROS CUANDO ES METODO DE PAGO TARJETA
        if (jsonparams.has("ssccr_types_of_credit_id")
            && jsonparams.has("Ssccr_Processor_Banck_ID")) {

          String strSsccrTypesOfCreditId = jsonparams.getString("ssccr_types_of_credit_id");
          String strSsccrProcessorBanckId = jsonparams.getString("Ssccr_Processor_Banck_ID");

          if (!strSsccrTypesOfCreditId.equals("null") && !strSsccrProcessorBanckId.equals("null")) {

            if (strAction.equals("PRD")) {

              String strFinancialAccountId = jsonparams.getString("fin_financial_account_id");
              FIN_FinancialAccount finAccount = OBDal.getInstance().get(FIN_FinancialAccount.class,
                  strFinancialAccountId);

              final OBCriteria<FIN_FinaccTransaction> obc = OBDal.getInstance()
                  .createCriteria(FIN_FinaccTransaction.class);
              obc.add(Restrictions.eq(FIN_FinaccTransaction.PROPERTY_FINPAYMENT, payment));
              obc.add(Restrictions.eq(FIN_FinaccTransaction.PROPERTY_ACCOUNT, finAccount));

              String strbanktransferId = jsonparams.getString("ssfi_banktransfer_id");
              ssfiBanktransfer objSsfiBanktransfer = OBDal.getInstance().get(ssfiBanktransfer.class,
                  strbanktransferId);

              SsccrTypesOfCredit objSsccrTypesOfCredit = OBDal.getInstance()
                  .get(SsccrTypesOfCredit.class, strSsccrTypesOfCreditId);

              SsccrProcessorBanck objSsccrProcessorBanck = OBDal.getInstance()
                  .get(SsccrProcessorBanck.class, strSsccrProcessorBanckId);

              String strC_Order_ID = jsonparams.getString("c_order_id");

              String strSsccrCardsTypesId = jsonparams.getString("Ssccr_Cards_Types_ID");
              SsccrCardsTypes objSsccrCardsTypes = OBDal.getInstance().get(SsccrCardsTypes.class,
                  strSsccrCardsTypesId);

              String strInvoiceNo = "";
              String strSalesOrderNo = "";
              Order objOrder = null;
              Invoice objInvoice = null;
              JSONObject orderInvoiceGrid = jsonparams.getJSONObject("order_invoice");
              JSONArray selectedPSDs = orderInvoiceGrid.getJSONArray("_selection");
              final OBCriteria<Invoice> objInvoiceLst = OBDal.getInstance()
                  .createCriteria(Invoice.class);

              if (strC_Order_ID == null || strC_Order_ID.equals("")
                  || strC_Order_ID.equals("null")) {
                for (int i = 0; i < selectedPSDs.length(); i++) {

                  JSONObject psdRow = selectedPSDs.getJSONObject(i);
                  strInvoiceNo = psdRow.getString("invoiceNo");
                  strSalesOrderNo = psdRow.getString("salesOrderNo");

                  if (strSalesOrderNo == null || strSalesOrderNo.equals("")) {
                    objInvoiceLst.add(Restrictions.eq(Invoice.PROPERTY_DOCUMENTNO, strInvoiceNo));
                  } else if (strInvoiceNo != null || !strInvoiceNo.equals("")) {
                    final OBCriteria<Order> objOrderLst = OBDal.getInstance()
                        .createCriteria(Order.class);
                    objOrderLst.add(Restrictions.eq(Order.PROPERTY_DOCUMENTNO, strSalesOrderNo));
                    objOrder = objOrderLst.list().get(0);
                    objInvoiceLst.add(Restrictions.eq(Invoice.PROPERTY_SALESORDER, objOrder));

                  }
                }
              } else {
                objOrder = OBDal.getInstance().get(Order.class, strC_Order_ID);
                objInvoiceLst.add(Restrictions.eq(Invoice.PROPERTY_SALESORDER, objOrder));
              }

              if (objInvoiceLst != null && !objInvoiceLst.equals("")) {
                if (selectedPSDs.length() > 0) {
                  for (Invoice invoiceobj : objInvoiceLst.list()) {
                    objInvoice = invoiceobj;
                    objOrder = invoiceobj.getSalesOrder();
                  }
                }
              }

              Calendar c = Calendar.getInstance();
              c.setTime(paymentDate);

              // Busqueda de Configuración conciliación de tarjetas
              if (objSsccrCardsTypes != null) {
                String value = new SimpleDateFormat("yyyy-MM-dd").format(paymentDate);
                ConnectionProvider conn = null;
                try {
                  conn = new DalConnectionProvider(true);
                  String strPaymentTermId = SSCCRPaymentTermData.paymentterm(conn,
                      objSsccrCardsTypes.getId(), objSsccrTypesOfCredit.getId(), value);
                  if (strPaymentTermId != null && !strPaymentTermId.equals("")) {
                    PaymentTerm objPaymentTerm = OBDal.getInstance().get(PaymentTerm.class,
                        strPaymentTermId);
                    c.add(Calendar.DAY_OF_YEAR,
                        objPaymentTerm.getOverduePaymentDaysRule().intValue());
                  } else {
                    OBDal.getInstance().rollbackAndClose();
                    JSONObject errorMessage = new JSONObject();
                    errorMessage.put("severity", "error");
                    errorMessage.put("text",
                        "No existe configuración de condición de pago para los parámetros dados.");
                    jsonResponse.put("retryExecution", openedFromMenu);
                    jsonResponse.put("message", errorMessage);
                    return jsonResponse;
                  }
                } catch (Exception ex) {
                  ex.printStackTrace();
                } finally {
                  conn.destroy();
                }
              }
              String lot = jsonparams.getString("Lote");

              for (FIN_FinaccTransaction financialAccount : obc.list()) {
                financialAccount.setSsccrSsfiBanktransfer(objSsfiBanktransfer);
                financialAccount.setSsccrTypesOfCredit(objSsccrTypesOfCredit);
                financialAccount.setSsccrProcessorBanck(objSsccrProcessorBanck);
                financialAccount.setSsccrCardsTypes(objSsccrCardsTypes);
                financialAccount.setSsccrCInvoice(objInvoice);
                financialAccount.setSsccrCOrder(objOrder);
                financialAccount.setSsccrFinPayment(payment);
                financialAccount.setSsccrLot(lot);
                if (objSsccrProcessorBanck != null) {
                  financialAccount.setSsccrExpirationDate(c.getTime());
                }
                if (jsonparams.get("reference_no") != JSONObject.NULL) {
                  financialAccount.setSsccrRecapno(jsonparams.getString("reference_no"));
                }
                try {
                  OBDal.getInstance().save(financialAccount);
                  OBDal.getInstance().flush();
                } finally {
                }
              }
            }

          }

        }
```

Esta implementacion permite procesar el pago y, si es con tarjeta y la acción es **PRD**, actualiza la transacción bancaria con la información de conciliación y guarda el número de referencia cuando está disponible.



Además, encontramos dentro de la V24 un pequeño cambio:

```java
  private BaseOBObject getAccountDimension(final JSONObject glItem, final String dimension,
      final Class<?> clazz) throws JSONException, ServletException {
    if (glItem.has(dimension) && glItem.get(dimension) != JSONObject.NULL) {
      final String dimensionId = glItem.getString(dimension);
      checkID(dimensionId);
      return (BaseOBObject) OBDal.getInstance().get(clazz, dimensionId);
    }
    return null;
  }
```

Esto básicamente, lee del JSON el ID de una “dimensión”, y si viene informado y no es null, lo valida y lo busca en la base de datos con `OBDal`. Si no hay datos, simplemente devuelve null.



La versión V18 es una personalización necesaria para el manejo de pagos con tarjeta. A diferencia de la versión estándar, permite capturar datos adicionales como el banco y el lote, calcula automáticamente cuándo debe ingresar el dinero a la cuenta y asegura que el pago quede correctamente asociado a la factura.

Para la V24, es importante integrar esta lógica fusionando el código, de modo que no se pierda la automatización del módulo de conciliación de tarjetas en ÁgilERP 24Q4.1.

**Es optimo implementarlo?**

Si se puede realizar la integracion ,ya que , si no se integra, los campos de tarjeta que se llenen en la pantalla se perderán al procesar el pago, dejando el módulo de conciliación de tarjetas sin datos.

* Se debe usar la v24 como base técnica (para mantener las mejoras de seguridad de Java 8 como los flujos `stream` y `Optional`) e inyectar manualmente los bloques de código `Ssccr` y las validaciones de referencia de la v18.



### Análisis comparativo: Estándar V18 vs Estándar V24

Análisis realizado sobre:

EstandarV18

> Modulo: ec.com.sidesoft.creditcard.reconciliation
>
> Clase: SSCCR\_AddPaymentActionHandler.java

EstandarV24

> Modulo: ec.com.sidesoft.creditcard.reconciliation
>
> Clase: SSCCR\_AddPaymentActionHandler.java

Lo que podemos ver es que en el ambiente Estándar V24 ya se encuentra integrada la customización que venía desde la V18. La diferencia principal es que ahora se aplican nuevas normas de seguridad, ya que en la versión anterior existía el riesgo de que, al abrir una conexión, se pudieran filtrar datos.

En esta versión Estándar 24, la apertura y el cierre de conexiones se manejan correctamente, lo que reduce ese riesgo y garantiza un mejor control.



