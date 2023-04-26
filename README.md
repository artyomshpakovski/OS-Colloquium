# OS-Colloquium

Предметной областью для моей реализации будет служить процессор платежей.

# State
Основной класс Payment, который предоставляет интерфейс для передачи данных и обрабатывает запросы на изменение состояния. Этот класс имеет переменную _state_ класса PaymentState, что дублирует интерфейс изменения состояния, определенного в классе Payment. Каждая операция PaymentState принимает экземпляр класса Payment как параметр, тем самым позволяя объетку PaymentState получить доступ к данным объекта Payment и изменить состояние платежа:
~~~ C++ class Payment {
public:
    Payment();
    Payment CreatePayment();
    void CheckAuthorization();
    void MakePayment();
    void CancelPayment();
private:
    int ID;
    int paymentSum;
    PaymentState* _state;
};
~~~

Класс состояния PaymentState
~~~ C++ class PaymentState {
public:
    virtual void EnterState(Payment*);
    virtual void ExitState(Payment*);
    virtual void AuthorizePayment(Payment*);
    virtual void ProcessPayment(Payment*);
protected:
    void ChangeState(Payment*, PaymentState*);

};
~~~

Три класса ребенка:
1) PaymentNew - описывает состояние только созданного платежа.
~~~ C++ class PaymentNew : public PaymentState {
    virtual void EnterState(Payment*);
    virtual void ExitState(Payment*);
    virtual void AuthorizePayment(Payment*);
    virtual void ProcessPayment(Payment*);
};
~~~
2) PaymentAuthorized - описывает состояние подтвержденного платежа.
~~~ C++ 
class PaymentAuthorized : public PaymentState {
    virtual void EnterState(Payment*);
    virtual void ExitState(Payment*);
    virtual void AuthorizePayment(Payment*);
    virtual void ProcessPayment(Payment*);
};
~~~
3) PaymentDeclined - описывает состояние отклоненного платежа.
~~~ C++ 
class PaymentDeclined : public PaymentState {
    virtual void EnterState(Payment*);
    virtual void ExitState(Payment*);
    virtual void AuthorizePayment(Payment*);
    virtual void ProcessPayment(Payment*);
};
~~~
Для каждого из этих классов описывается функция своим образом, в зависимости от текущего состояния объекта.
# Fabric Method
Для данного случая буду реализовывать фабричный паттерн для создания объектов состояния.
~~~C++
class PaymentStateFactory {
public:
    virtual ~PaymentStateFactory() {}
    virtual PaymentState* CreatePayment() = 0;
};
~~~
Фабрика для PaymentNew
~~~C++
class PaymentNewFactory : public PaymentStateFactory {
public:
    PaymentState* CreatePayment() override {
        return new PaymentNew();
    }
};
~~~
Фабрика для PaymentAuthorized
~~~C++
class PaymentAuthorizedFactory : public PaymentStateFactory {
public:
    PaymentState* CreatePayment() override {
        return new PaymentAuthorized();
    }
};
~~~
Фабрика для PaymentDeclined
~~~C++
class PaymentDeclinedFactory : public PaymentStateFactory {
public:
    PaymentState* CreatePayment() override {
        return new PaymentDeclined();
    }
};
~~~
# UnitTest

~~~C++

// Tests for the PaymentNew class
TEST(PaymentNewTest, EnterStateTest) {
    Payment payment;
    PaymentNew payment_new;
    payment_new.EnterState(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_new);
}

TEST(PaymentNewTest, ExitStateTest) {
    Payment payment;
    PaymentNew payment_new;
    payment_new.EnterState(&payment);
    payment_new.ExitState(&payment);

    EXPECT_EQ(payment.GetCurrentState(), nullptr);
}

TEST(PaymentNewTest, AuthorizePaymentTest) {
    Payment payment;
    PaymentNew payment_new;
    payment_new.AuthorizePayment(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_new);
}

TEST(PaymentNewTest, ProcessPaymentTest) {
    Payment payment;
    PaymentNew payment_new;
    payment_new.ProcessPayment(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_new);
}

// Tests for the PaymentAuthorized class
TEST(PaymentAuthorizedTest, EnterStateTest) {
    Payment payment;
    PaymentAuthorized payment_authorized;
    payment_authorized.EnterState(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_authorized);
}

TEST(PaymentAuthorizedTest, ExitStateTest) {
    Payment payment;
    PaymentAuthorized payment_authorized;
    payment_authorized.EnterState(&payment);
    payment_authorized.ExitState(&payment);

    EXPECT_EQ(payment.GetCurrentState(), nullptr);
}

TEST(PaymentAuthorizedTest, AuthorizePaymentTest) {
    Payment payment;
    PaymentAuthorized payment_authorized;
    payment_authorized.AuthorizePayment(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_authorized);
}

TEST(PaymentAuthorizedTest, ProcessPaymentTest) {
    Payment payment;
    PaymentAuthorized payment_authorized;
    payment_authorized.ProcessPayment(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_authorized);
}

// Tests for the PaymentDeclined class
TEST(PaymentDeclinedTest, EnterStateTest) {
    Payment payment;
    PaymentDeclined payment_declined;
    payment_declined.EnterState(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_declined);
}

TEST(PaymentDeclinedTest, ExitStateTest) {
    Payment payment;
    PaymentDeclined payment_declined;
    payment_declined.EnterState(&payment);
    payment_declined.ExitState(&payment);

    EXPECT_EQ(payment.GetCurrentState(), nullptr);
}

TEST(PaymentDeclinedTest, AuthorizePaymentTest) {
    Payment payment;
    PaymentDeclined payment_declined;
    payment_declined.AuthorizePayment(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_declined);
}

TEST(PaymentDeclinedTest, ProcessPaymentTest) {
    Payment payment;
    PaymentDeclined payment_declined;
    payment_declined.ProcessPayment(&payment);

    EXPECT_EQ(payment.GetCurrentState(), &payment_declined);
}
~~~
Здесь используется не описанный раннее GetCurrentState(), который возвращает тип класса. Я привел только пример неполного покрытия тестами, поскольку полное покрытие будет довольно объемным.
