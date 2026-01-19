# Documentacion Modulo org.openbravo.advpaymentmngt



> org.openbravo.advpaymentmngt

Cambios que se realizaron en este modulo, especificamente en  estos 2 modulos

> ec.com.sidesoft.custom.advpaymentmngt.actionHandler
>
> org.openbravo.advpaymentmngt.actionHandler



**Se agrego esta linea**&#x20;

```java
transaction.setLineNo(TransactionsDao.getTransactionMaxLineNo(account) + 10);
```

Lo que hace es colocar a la transaccion un numero de linea, luego la linea mayor es decir la ultima y la suma +10, digamos si le coloco a la transaccion 40 el nuveo numero de quedaria 50

**Ademas se agrego el siguiente metodo**

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

Ademas se encontro&#x20;

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

