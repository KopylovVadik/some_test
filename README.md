* Пункт 1

Для решения данной задачи я бы применил абстрактные модели django.

    class AbstractState(models.Model):
    
    class Meta:
        abstract = True

    @classmethod
    def get(cls, state_id):
        return cls.objects.get(pk=state_id)class 


    DeliveryState(AbstractState):

    class Meta:
        verbose_name = "Состояние доставки"
        verbose_name_plural = "Состояния доставок"

    STATE_NEW = 1  # Новая
    STATE_ISSUED = 2  # Выдана курьеру
    STATE_DELIVERED = 3  # Доставлена
    STATE_HANDED = 4  # Курьер сдал
    STATE_REFUSED = 5  # Отказ
    STATE_PAID_REFUSED = 6  # Отказ с оплатой курьеру
    STATE_COMPLETE = 7  # Завершена
    STATE_NONE = 8  # Не определено


Соответсвенно, теперь сможем работать так:

`
delivery_state = DeliveryState.get(DeliveryState.STATE_NEW)
` 

* Пункт 2

Для решения данной задачи используем сигналы pre-save и pst-save.



    @receiver(pre_save, sender=Lead)
        def validate_lead_state_transition(sender, instance, **kwargs):
            if instance.pk:
                # Получаем предыдущее состояние объекта из базы данных
                previous_state = Lead.objects.get(pk=instance.pk).state_id
        
        # Определяем возможные переходы между состояниями
        valid_transitions = {
            LeadState.STATE_NEW: [LeadState.STATE_IN_PROGRESS],
            LeadState.STATE_IN_PROGRESS: [LeadState.STATE_POSTPONED, LeadState.STATE_DONE],
            LeadState.STATE_POSTPONED: [LeadState.STATE_IN_PROGRESS, LeadState.STATE_DONE],
            LeadState.STATE_DONE: []
        }
        
        # Проверяем, является ли запрошенный переход допустимым
        if instance.state_id not in valid_transitions[previous_state]:
            raise ValueError("Недопустимый переход состояния")

    @receiver(post_save, sender=Lead)
    def perform_lead_state_transition(sender, instance, **kwargs):
        if instance.pk:
            # Получаем предыдущее состояние объекта из базы данных
            previous_state = Lead.objects.get(pk=instance.pk).state_id
        
        # Определяем действия, связанные с каждым переходом состояния
        state_actions = {
            (LeadState.STATE_NEW, LeadState.STATE_IN_PROGRESS): instance.method1,
            (LeadState.STATE_IN_PROGRESS, LeadState.STATE_POSTPONED): instance.method2,
            (LeadState.STATE_IN_PROGRESS, LeadState.STATE_DONE): instance.method3,
            (LeadState.STATE_POSTPONED, LeadState.STATE_IN_PROGRESS): instance.method4,
            (LeadState.STATE_POSTPONED, LeadState.STATE_DONE): instance.method5
        }
        
        # Выполняем соответствующее действие для выполненного перехода состояния
        if instance.state_id != previous_state and (previous_state, instance.state_id) in state_actions:
            action = state_actions[(previous_state, instance.state_id)]
            action()

    def method1(self):
        # Дополнительная бизнес логика для перехода из "Новый" в "В работе"
        pass

    def method2(self):
        # Дополнительная бизнес логика для перехода из "В работе" в "Приостановлен"
        pass

    def method3(self):
        # Дополнительная бизнес логика для перехода из "В работе" в "Завершен"
        pass

    def method4(self):
        # Дополнительная бизнес логика для перехода из "Приостановлен" в "В работе"
        pass

    def method5(self):
        # Дополнительная бизнес логика для перехода из "Приостановлен" в "Завершен"
        pass

* Пункт 3

В данном коде убрал не нужные проверки, добавил тразакционность, проверку баланса пользователя перед списанием средтсв, обновление статуса товара (т.к.товар уникален).
` 

    @classmethod
    def buy(cls, user, item_id):
        with transaction.atomic():
            product = Product.objects.select_for_update().get(item_id=item_id)
                if product.available and user.balance >= product.price:
                    try:
                        # списание средств со счета пользователя
                        user.withdraw(product.price)
                    
                    # информация о купленном товаре
                    send_email_to_user_of_buy_product(user)
                    
                    # обновление состояния товара
                    product.available = False
                    product.buyer = user
                    product.save()
                    
                    return True
                except Exception as e:
                    # обработка возможных исключений при списании средств или отправке почты
                    # логирование ошибки
                    return False
            else:
                return False
        else:
            return False

* Пункт 4



    def print_tariff_info(tariff_info, indent=0):

        for key, value in tariff_info.items():
            if isinstance(value, dict):
                print(' ' * indent + key)
                print_tariff_info(value, indent + 2)
            elif isinstance(value, list):
                print(' ' * indent + key)
                for item in value:
                    if isinstance(item, dict):
                        print_tariff_info(item, indent + 2)
                    else:
                        print(' ' * (indent + 2) + item)
            else:
                print(' ' * (indent + 2) + key)
    
    print_tariff_info(tariff_info['root'])`
