# 1. Android code style

## 1.1 Правила именования

### 1.1.1 Классы
Именна классов пишем в [UpperCamelCase](http://en.wikipedia.org/wiki/CamelCase).

Для классов-компонентов Android в названии должен быть отражен компонент: `SignInActivity`, `SignInFragment`, `ImageUploaderService`, `ChangePasswordDialog`.
Ресурсы именуем в __lowercase_underscore__.

### 1.1.2 Layout файлы

В качестве суффикса используем имя компонента для которого предусмотрен этот layout, например: 

| Компонент        | Имя класса             | Имя файла разметки            |
| ---------------- | ---------------------- | ----------------------------- |
| Activity         | `UserProfileActivity`  | `activity_user_profile.xml`   |
| Fragment         | `SignUpFragment`       | `fragment_sign_up.xml`        |
| Dialog           | `ChangePasswordDialog` | `dialog_change_password.xml`  |
| RecyclerView item| ---                    | `item_person.xml`             |
| Partial layout   | ---                    | `partial_channel_info.xml`    |

Обратите внимание на 2 последних примера-исключения: первый применим для recycler view, adapter view и подобных, когда описываем часть списочных структур, второй - для переиспользуемой разметки.

### 1.1.2 Переменные

* Переменные именуем, используя lowerCamelCase (с некоторыми исключениями, о которых далее)
* Приватные статические поля именуются с префиксом __s__.
* Статические константные поля именуем ALL_CAPS_WITH_UNDERSCORES.

Например:

```java
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int packagePrivate;
    private int fieldPrivate;
    protected int fieldProtected;
}
```

### 1.1.3 Ресурсы

Имена ресурсов пишем в __lowercase_underscore__.

### 1.1.4 Идентификаторы

Также именуем в __lowercase_underscore__. В качестве префикса используем имя элемента, например:

| Элемент            | Префикс            |
| -----------------  | ----------------- |
| `TextView`           | `text_`             |
| `ImageView`          | `image_`            |
| `Button`             | `button_`           |
| `Menu`               | `menu_`             |

```xml
<ImageView
    android:id="@+id/image_profile"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

```xml
<menu>
	<item
        android:id="@+id/menu_done"
        android:title="Done" />
</menu>
```

### 1.1.5 Строковые ресурсы

Для контекстозависимых строк в качестве префикса используем название этого контекста, например `registration_email_hint` or `registration_name_hint`. Если это какие-то общие строки, следуем следующим правилам:


| Префикс             | Описание                           |
| -----------------  | --------------------------------------|
| `error_`             | Сообщение об ошибке                |
| `msg_`               | Информационное сообщение        |
| `title_`             | Заголовок, например, диалога     |
| `action_`            | Действие, например, "Сохранить"  |

### 1.1.6 Коммиты

Лучшее название коммита - тезисное отражение проделанной работы. В начале лучше написать номер бага. Если есть какие-то особые указания (например неявная правка или костыль для платформы) пишем их через разрыв строки.

## 2.1 Форматирование кода

### 2.1.1 Используем пробелы для форматирования

Используем __4 пробела__ для форматирования отступа блоков кода:

```java
if (x == 1) {
    x++;
}
```

Используем __8 пробелов__ для разрыва строк:

```java
Instrument i =
        someLongExpression(that, wouldNotFit, on, one, line);
```

### 2.1.2 Перенос скобок

Скобки никогда не переносятся:

```java
class MyClass {
    int func() {
        if (something) {
            // ...
        } else if (somethingElse) {
            // ...
        } else {
            // ...
        }
    }
}
```

Несмотря на то, что в AndroidStudio мы можем выбирать куда поставить точку останова в случае условия и тела на одно строке, мы не хотим в случае изменений чего-либо одного становиться авторами изменений всей строки, поэтому следующее __плохо__:

```java
if (condition) body();
```

```java
if (condition)
    body();
```

### 2.1.3 Длина строк и методов

Максимальная длина строки не должна превышать 120 символов.

Методы должны быть максимально короткими и выполнять ту задачу, которая перед ними поставлена (обычно 
отражается в названии). Так как длинные методы иногда целесообразны, ограничений по длине нет, но если метод 
переваливает за ~40 строк, задумайтесь!

## 2.2 Code guidlines

### 2.2.1 Исключения не игнорируются

Никогда не делайте так:

```java
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) { }
}
```

Даже если вы абсолютно уверены, что никогда не словите подобное исключение - всегда обрабатывайте исключение!
Но, если же бизнес-логика диктует или код устроен так, что исключение может быть проигнорировано, комментируйте почему проигнорировать - хорошая идея.

```java
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) { 
		// Method is documented to just ignore invalid user input.
        // serverPort will just be unchanged.
	}
}
```

### 2.2.2 Ловите конкретные исключения

Никогда не делайте так:

```java
try {
    someComplicatedIOFunction();        // may throw IOException
    someComplicatedParsingFunction();   // may throw ParsingException
    someComplicatedSecurityFunction();  // may throw SecurityException
    // phew, made it all the way
} catch (Exception e) {                 
    handleError();                      
}
```

### 2.2.3 Никогда не используем finalize-методы

Закрыть какой-нибудь дескриптор в методе finalize() может показаться отличной идеей (особенно полсе плотной работы с C/C++), но это не так. Мы не знаем, когда он может быть вызван и будет ли вызван вообще. Если это необходимо - создавайте метод close().

### 2.2.4 Аннотации

#### 2.2.4.1 Правила использования

* `@NonNull`: Указываем всегда. Без необходимости оставлять полю возможность зануления не стоит. Это облегчит поддержку и уменьшит количество NPE.
* `@Override`: Используем всегда, исключений нет.
* `@SuppressWarnings`: Используем только тогда, когда другого выхода нет, например, глушим unused предупреждения для методов, принимающих события от подписчиков.

#### 2.2.4.2 Стиль применения

__Классы, методы или конструкторы__

Аннотация на строку, например:

```java
@AnnotationA
@AnnotationB
public class MyAnnotatedClass { }
```

__Поля класса__

Умещается в ограничение 1 строки - пишем все на 1 строке

```java
@Nullable @Mock DataManager mDataManager;
```

### 2.2.5 Ограничение области видимости переменных

Объявляем переменные ровно в тот момент, когда они нам нужны. Объявили - тут же инициализируем. Если мы не знаем чем инициализировать переменную - она нам еще не нужна! Единственное исключение (по инициализированию) - блок try-catch. 
Если переменная нужна внутри блока, инстанцируем ее перед. Все переменные для циклов объявляем в операторе по возможности.

### 2.2.6 Логирование

Логируем исключительно через Timber. 
Verbose и debug исключаются из релизных сборок, info конкатенируется до первой ошибки, error логирует в crashlytics, поэтому лишнего и бессмысленного в .i и .e не логируем.

### 2.2.7 Порядок полей и методов

1. Константы
2. Перечисления
3. Поля
4. Конструкторы
5. Методы жизненного цикла
6. Переопределенные методы и коллбеки
7. Публичные методы
8. Приватные методы
9. Вложенные классы

Пример:

```java
public class MainActivity extends Activity {

    private static final String TAG = MainActivity.class.getSimpleName();

    private String title;
    private TextView textViewTitle;
    
    @Override
    public void onCreate() {
        ...
    }

    public void setTitle(String title) {
    	this.title = title;
    }

    private void setUpView() {
        ...
    }

    static class AnInnerClass {

    }

}
```

Если перечислениям, коллбекам и классам по смыслу не обязательно быть вложенными (в случае классов - не нужен доступ к полям внешнего класса и можно обойтись внедрением зависимости) - выделяем в отдельный файл.

### 2.2.8 Порядок аргументов функций

Стандартный подход - в контекстозависимых функций __контекст__ всегда указывается __первым__ аргументом.
Для __коллбеков__ - все с точностью наоборот. Их указвыаем всегда __последними__.

Пример:

```java
public User loadUser(Context context, int userId);

public void loadUserAsync(Context context, int userId, UserCallback callback);
```

### 2.2.9 Аргументы для фрагментов и активностей

Никогда не создаем собственных конструкторов для Fragment и Activity!
Если для создания нужны аргументы - создаем public static метод, который конструирует на основании переданных аргументов Intent или Fragment.
Как правило, для активностей такие методы называет getStartIntent(), для фрагментов - getNewInstance():

```java
public static Intent getStartIntent(Context context, User user) {
	Intent intent = new Intent(context, ThisActivity.class);
	intent.putParcelableExtra(EXTRA_USER, user);
	return intent;
}
```

```java
public static UserFragment newInstance(User user) {
	UserFragment fragment = new UserFragment();
	Bundle args = new Bundle();
	args.putParcelable(ARGUMENT_USER, user);
	fragment.setArguments(args)
	return fragment;
}
```

### 2.2.10 Правила переноса строк

Четких правил здесь нет, но стоит придерживаться следующего:

__Перенос на операторе__

Перенос строки делаем __перед__ оператором. Например:

```java
int longName = someLongStateVariable == packet.enum.SomeNotInitializedState
		? packet.enum.Initialized
		: packet.enum.Restarted
```

__Приравнивание__ - исключение

```java
int longName =
        anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne + theFinalOne;
```

__Цепочки вызовов (method chaining)__

Каждый вызов метода - с новой строки. Метод начинается с точки:

```java
new Builder().with(context)
        .addListener(new Listener())
        .retry(3)
		.build()
```

## 2.3 Правила XML

### 2.3.1 Использование закрывающего тега

Если у элемента нет контента, __всегда__ закрываем тег сразу.

Так делать хорошо:

```xml
<TextView
	android:id="@+id/text_view_profile"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content" />
```

А так - __плохо__ :

```xml
<TextView
    android:id="@+id/text_view_profile"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" >
</TextView>
```

# 3 Общие положения и best practicies

## 3.1 Несоответствия со стилем

Увидел - исправляй, чтобы соответствовало

## 3.2 Коммиты

Стараемся сделать так, чтобы в каждом коммите было минимальное количество изменений. Не коммитим правку и рефакторинг в одном чейнджсете, разбиваем. Если делаем большую фичу, коммиты должны быть итеративные, показывающие развитие фичи.

## 3.3 Ревью

Как следствие предыдущего пункта и нашего ревьюборда составные зависимые коммиты заливаем поступательно. То есть цепочка зависимых 5 коммитов на ревью выглядит как 5 изменений 1 коммита, чтоыб можно было на ревью рассмотреть каждый в отдельности. Правки вносим уже поверх.