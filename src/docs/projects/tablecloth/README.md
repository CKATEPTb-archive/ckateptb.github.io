# Tablecloth

ссылки: [GitHub](https://github.com/CKATEPTb/Tablecloth), [Wiki](/projects/tablecloth/)

Это вспомогательный плагин для других моих плагинов.

## Spring

Если вы не знакомы со Spring, вы можете ознакомиться с ним в [официальной документации](https://spring.io).

Но давайте мы сэкономим ваше время, и я расскажу вам об этом в двух словах.

Одна из основных особенностей среды Spring - это контейнер IoC (Inversion of Control). IoC-контейнер отвечает за управление объектами приложения.

Это делает нашу разработку проще, быстрее и красивее, почему бы нам не использовать это при разработке плагинов?

### Особенности
* Полная поддержка контейнера Spring
* Автоматическая регистрация событий в компонентах реализованных как Listener
* Spring `@Scheduler` работает с `initialDelay` и `fixedRate` на основе тиков Minecraft
* Невероятно прост в использовании

### Введение
Наличие Spring абсолютно ничего не даст простому пользователю, но если вы разработчик, то для вас будут открыты все возможности [AnnotationConfigApplicationContext](https://www.baeldung.com/spring-application-context)

#### Теперь вы можете перейти непосредственно к вашему плагину
Все, что нам нужно сделать, это расширить наш класс плагина как SpringPlugin.

```java
public class MyPlugin extends SpringPlugin {
}
```

Если по какой-то причине структура вашего плагина не соответствует стандартам, вам необходимо вручную указать родительский пакет для сканирования классов, тогда основной класс вашего плагина будет выглядеть так

```java
public class MyPlugin extends AbstractSpringContextHolder {
    public MyPlugin() {
        SpringContext.register(this, "com.example.mypluginpackage");
    }
}
```

Но я надеюсь, что основной класс вашего плагина уже находится в родительском пакете.

Пример правильной структуры:

```
    > com.example.myplugin      << PARENT PACKAGE
        > firstawesomepacket    << JUST PACKAGE
            AwesomeClass        << JUST CLASS
        > secondawesomepacket   << JUST PACKAGE
        MyPlugin                << PLUGIN CLASS
```

Пример неправильной структуры:

```
    > com.example.myplugin      << PARENT PACKAGE
        > firstawesomepacket    << JUST PACKAGE
            AwesomeClass        << JUST CLASS
        > secondawesomepacket   << JUST PACKAGE
            MyPlugin            << PLUGIN CLASS
```

Все, ничего сложного здесь нет. Теперь вы можете использовать Spring в своем плагине.

Давайте поверхностно разберемся, что такое Spring и с чем его едят.
Страшные аббревиатуры, такие как IoC и DI, которые вы возможно встречали — это все про него, про Spring!
Начнем с самых истоков org.springframework.context.ApplicationContext, далее просто контейнер. Он создает и хранит экземпляры ваших классов. Многие из вас даже не поняли, что это значит, но это не повод переставать читать прямо сейчас! Вы поймете это по ходу, на базовых примерах.
Для того чтобы Spring создал контейнер с нашими экземплярами, ему нужно знать из каких (классов/объектов) будет состоять ваше приложение, как они создаются и какие у них есть зависимости.

#### Какие бывают контейнер и как их создать

У интерфейса ApplicationContext есть большое количество реализаций:
* ClassPathXmlApplicationContext
* FileSystemXmlApplicationContext
* GenericGroovyApplicationContext
* AnnotationConfigApplicationContext
* и даже StaticApplicationContext
* а также некоторые другие.

Современным способом конфигурирования считаются аннотации (AnnotationConfigApplicationContext), и мы используем именно их.

С данным плагином, у вас нет необходимости создавать контейнер, ведь он уже создан в нем. Если вы действительно хотите знать, как его создать, в интернете полно информации по этому поводу.

#### В двух словах про контейнер
Под словом «Spring» обычно подразумевают просто IoC-контейнер, помогающий структурировать Java-приложения. Но вы должны знать, что в действительности под словом «Spring» скрывается целый мир.

IoC-контейнер, это замечательный способ собрать приложение «по кусочкам» из разных компонентов. Spring предоставляет удобные способы как написания данных «кусочков», так и объединения их в единое приложение.

Например, у нас есть два класса:

Сервис:
```java
public class MyService {

    private ServiceDependency dependency;

    public MyService(ServiceDependency dependency) {
        this.dependency = dependency;
    }

    public void setDependency(ServiceDependency dependency) {
        this.dependency = dependency;
    }

    public ServiceDependency getDependency() {
        return this.dependency;
    }

    public void usefulWork() {
        this.dependency.dependentWork();
    }
}
```
И его зависимость:
```java
public class ServiceDependency {
    // fields

    public ServiceDependency() {
    }

    public void dependentWork() {
        // any actions
    }
}
```
Самый простой способ объединить эти компоненты в единое приложение – это написать что-то вроде:
```java
public class MyClass {
    public MyClass() {
        ServiceDependency dep = new ServiceDependency();
        MyService service = new MyService(dep);
        service.usefulWork();
    }
}
```
Несмотря на простоту, данный код обладает серьезными недостатками, которые являются критическими для больших проектов. Действительно, в данном примере вполне очевидно, что экземпляр класса ServiceDependency необходимо создавать раньше, чем экземпляр объекта MyService. А в больших проектах таких сервисов и зависимостей может быть столько, что перебор программистом порядка создания объектов занимал бы совсем неприличное время.

Лично мне хотелось бы экономить свое время и не делать то, что по факту можно не делать! Хотелось бы даже не задумываться о создании объектов, их порядке и прочем.

Здесь и приходит на помощь Spring, а если быть точнее, то Spring Context. Вместе со Spring, нам в этой задаче приходит на помощь Lombok.

Если в вашем проекте, по какой-то причине все еще нет Lombok, то давайте добавим его прямо сейчас!
```groovy
// Добавляем Lombok в наш проект
dependencies {
    compileOnly 'org.projectlombok:lombok:1.18.12'
    annotationProcessor 'org.projectlombok:lombok:1.18.12'
}
```
Давайте посмотрим на примере, как Spring и Lombok делают нашу жизнь проще!

Модифицируем немного наши классы, добавив так называемые аннотации стереотипов.

```java
@Getter
@Setter
@Service
@AllArgsConstructor
public class MyService {
    private ServiceDependency dependency;

    public void usefulWork() {
        this.dependency.dependentWork();
    }
}
```
Сейчас я расскажу вам об аннотациях, которые мы использовали в нашем сервисе.
* `@Service` - Сообщает в Spring, что это класс сервис, а тот в свою очередь автоматически создаст и сохранит его объект в контейнере
* `@AllArgsConstructor` - Это уже аннотация из Lombok, которая автоматически создает конструктор, со всеми переменными объявленными в нашем классе
* `@Getter` - Говорит что у всех, объявленных в нашем классе, переменных должен быть метод `get`
* `@Setter` - Аналогично `@Getter`, только с методом `set`

По итогу на выходе мы получим такой же класс, как указали выше, но написав на порядок меньше строк. Отлично!

```java
@Component
@NoArgsConstructor
public class ServiceDependency {
    public void dependentWork() {
        // any actions
    }
}
```

Теперь рассмотрим эти аннотации
* `@Component` - Сообщает в Spring, что это компонент для нашего сервиса и тот поведет себя так же, как и с аннотацией `@Service`
* `@NoArgsConstructor` - Это тоже аннотация из Lombok, но она же, создает пустой конструктор

Ну вот, ситуация повторилась, мы опять получили необходимый нам результат, написав на порядок меньше строк.

Ну и конечно, класс, который должен обработать этот сервис, в нашем случае, это класс нашего плагина:

```java
public class MyPlugin extends SpringPlugin {
    public MyPlugin() {
        getContext().getBean(MyService.class).usefulWork();
    }
}
```

И все! Обратите внимание, что здесь не написано ни одно new для наших объектов.

Также, хочу добавить, что аннотации @Component, @Repository, @Controller, @Configuration На практике для вас будут иметь одинаковый эффект, по этому нет разницы как вы аннотируете ваш класс. Это сделано, чтобы вы понимали, что к чему.

По логике:
* @Service - что этот класс — сервис для чего-то
* @Repository - читает/записывает информацию (например из/в файл)
* @Controller - контролирует запросы
* @Configuration - настройка нашего приложение
* @Component - ну... Если все остальное не подошло, то используйте эту аннотацию

Но на деле, все это вы реализуете сами, по этому можете пометить класс как вашей душе угодно.

Все перечисленные аннотации унаследованы от аннотации @Component, а аннотированные ими классы принято называть компонентами

#### Как добавить в контейнер уже существующий объект, который был создан из вне?

Тут тоже нет ничего сложного и есть целых два решения!

Первое и я бы сказал правильное решение звучит так:

Необходимо создать вспомогательный класс конфигурацию нашего приложения с аннотацией @Configuration, если такого еще нет. Затем в нем мы создаем метод, который возвращает необходимый нам объект и аннотируем его как @Bean И все, наша потребность удовлетворена!

Давайте посмотрим на примере, допустим мы хотим сохранить объект другого плагина в контейнер, чтобы в дальнейшем автоматизировать работу с ним.

```java
@Configuration
@NoArgsConstructor
public class MyConfiguration {
    @Bean
    public OtherPlugin getOtherPlugin() {
        return OtherPlugin.getInstance();
    }
}
```

Вот и все, теперь мы можем использовать это в контейнере, чтобы автоматизировать рутинное написание кода

Давайте поставим перед собой задачу добавить сервер в наш контейнер, вот как мы это сделаем:
```java
@Configuration
@NoArgsConstructor
public class ServerRegistry {
    public ServerRegistry() {
        Server server = Bukkit.getServer();
        SpringContext.getInstance().getBeanFactory().registerResolvableDependency(server.getClass(), server);
    }
}
```

#### Как получить доступ к объекту контейнера из вне?
`SpringContext.getInstance().getBean(Clazz.class)`, где `Clazz.class` - это класс объекта, который мы хотели бы получить.

#### Как отличать несколько объектов одного класса

Этот вопрос мы так-же рассмотрим на примере. По-умолчанию в Minecraft есть 3 мира, это normal, nether и end, допустим мы каждый из них хотим добавить в контейнер, но все миры это объекты World. Тут нам понадобиться уже знакомый нам вспомогательный класс конфигурации

```java
@Configuration
@NoArgsConstructor
public class MyConfiguration {
    @Bean("normal")
    public World getNormalWorld() {
        return normal;
    }

    @Bean("nether")
    public World getNetherWorld() {
        return nether;
    }

    @Bean("end")
    public World getEndWorld() {
        return end;
    }
}
```

Теперь нам нужно указать, какой именно мир нам необходим. Допустим мы хотим получить nether.

```java
@Getter
@Setter
@Component
public class NetherWorldHolder {
    private final World nether;

    public NetherWorldHolder(@Qualifier("nether") World nether) {
        this.nether = nether;
    }
}
```

#### Как зарегистрировать событие в компоненте

Помимо того, что вы можете сделать это, как заявлено в Bukkit, вы можете сделать это расширив ваш компонент интерфейсом Listener. Это даст вам возможность быстро зарегистрировать события.

```java
@Component
public class Handler implements Listener {
    @EventHandler
    public void on(Event event) {
        // TODO
    }
}
```

#### Как использовать Scheduler о котором говорилось в особенностях

Выполнять какой-то алгоритм с определенным интервалом или с задержкой — классика! У нас есть удобное решение для этого, заточенное под тики Minecraft Для этого, нужно просто аннотировать метод, который необходимо повторять, как @Schedule и в тиках указать задержку перед выполнением initialDelay или интервал fixedRate. Вы можете указать оба параметра, тогда initialDelay сработает только перед первым выполнением

```java
@Service
@NoArgsConstructor
public class MyService {
    // (20 тиков = 1 секунда)
    // Данный метод будет выполнен каждую секунду спустя 2 секунды
    @Schedule(initialDelay = 40, fixedRate = 20)
    public void processEverySecondWithTwoSecondInitialDelay() {
        //...
    }
}
```

Вы поверхностно ознакомились с основами Spring, а это только база, как уже упоминалось ранее, под словом «Spring» скрывается целый мир! Если у вас есть желание изучить этот мир, то в интернете полно материала по одному из самых популярных java framework'ов

Spring настолько большой, что его попросту нереально описать тут, но этого хватит, чтобы вы могли начать!

## Collider

Бывают такие моменты, когда нам просто не достаточно стандартных AABB, которые присутствуют в Minecraft, для этого вы можете использовать наши собственные системы коллизий

Создав коллизию, вы можете установить ее в любую локацию используя `Collider#at`

После чего вы можете:
* Выполнить действия с Entity, которые затрагивают наш Collider
  ```
    new SphereCollider(World, Location, Radius).handleEntityCollision(boolean livingOnly, boolean armorStandCollision, EntityCollisionCallback callback, Predicate<Entity> filter)
    ```
* Выполнить действия с Block, которые затрагивают наш Collider
  ```
    new SphereCollider(World, Location, Radius).handleBlockCollisions(boolean ignorePassable, boolean ignoreLiquids, BlockCollisionCallback callback, Predicate<Block> filter)
    ```
* Выполнить действия с Location, которые затрагивают наш Collider
  ```
    new SphereCollider(World, Location, Radius).handlePositionCollisions(double step, PositionCollisionCallback callback)
    ```

### Примеры
* [Sphere](https://github.com/CKATEPTb/Avatar-The-Legend-of-Korra/blob/master/src/main/java/ru/ckateptb/abilityslots/avatar/air/ability/AirBlast.java#L90)
* [Disk](https://github.com/CKATEPTb/Avatar-The-Legend-of-Korra/blob/master/src/main/java/ru/ckateptb/abilityslots/avatar/air/ability/AirBlade.java#L91)
* [Ray](https://github.com/CKATEPTb/AbilitySlots/blob/master/src/main/java/ru/ckateptb/abilityslots/entity/AbilityTargetLiving.java#L521)
* [AABB](https://github.com/CKATEPTb/Tablecloth/blob/master/src/main/java/ru/ckateptb/tablecloth/util/WorldUtils.java#L23)
* [OBB](https://github.com/CKATEPTb/Tablecloth/blob/9347820dea962c4480917189159e58f276e42ed1/src/main/java/ru/ckateptb/tablecloth/command/TableclothCommand.java#L107)

## Yaml конфигурация на основе аннотаций
Чтобы ускорить создание конфигураций, достаточно создать класс компонент и расширить его как YamlConfig

После чего, любая переменная, которая отмечена аннотацией @ConfigFiled будет вынесена в конфигурацию.

Важно! Не присваивайте переменной модификатор final!
### Пример
```java
@Getter
@Component
public class TableclothConfig extends YamlConfig {
    @ConfigField(name = "debug.collider", comment = "This function is necessary for debugging collisions (do not touch if you do not understand what it is about)")
    private boolean debugCollider = false;

    @ConfigField(name = "paralyze.name")
    private String paralyzeName = "§b§l<==§2§l Paralyzed §b§l==>";
    @ConfigField(name = "paralyze.type", comment = "Allowed types INVENTORY (Not perfect, but very light), ARMORSTAND (recommended ), MOVE_HANDLER (May cause TPS drawdowns with a large number of instances)")
    private String paralyzeType = ParalyzeType.ARMORSTAND.name();

    public ParalyzeType getParalyzeType() {
        return ParalyzeType.valueOf(paralyzeType);
    }

  @Override
  public Plugin getPlugin() {
    return Tablecloth.getInstance();
  }
}
```

## GUI

Мало кому нравиться управление плагином через чат, все хотят интерактивное GUI, и я предлагаю вам следующее

* [Chest](https://github.com/CKATEPTb/LoreImprove/blob/c9217407cead148cd18bb79c717ae3958427d737/src/main/java/ru/ckateptb/loreimprove/gui/AttributeManagerGui.java#L40)
* [Anvil](https://github.com/WesJD/AnvilGUI)
* [Book](https://github.com/upperlevel/book-api)

## ImmutableVector

Это расширенная версия Bukkit Vector, которая имеет ряд дополнительных методов. Особенность ImmutableVector в том, что при изменении вы получаете новый экземпляр, а старый остается без изменений.

## Particle

Собственная реализация Particle, которая упрощает вам их рендер в мире или для конкретных игроков.

## Temporary

Еще одной особенностью является перечень временных механик, такие как
* TemporaryBlock - Устанавливает временный блок
* TemporaryParalyze - Временно парализует LivingEntity
* TemporaryFallingBlock
* и т.д. [список](https://github.com/CKATEPTb/Tablecloth/tree/master/src/main/java/ru/ckateptb/tablecloth/temporary)