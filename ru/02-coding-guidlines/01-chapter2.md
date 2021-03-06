# Часть 2. Требования к стилю программирования. #

## 2.1. Общие замечания #

**[2.1.1]. см. пункт [1.1.1]**

**[2.1.2]. см. пункт [1.1.2]**

## 2.2. Миграция из С в С++ ##

**[2.2.1] Следует избегать инструкций #define**

Пример:

    const double PI = 3.1415; // NOT: #define PI 3.1415

    template < class T >
    const T& max( const T& a, const T& b ) 
    { 
        return ( a > b ) ? a : b; 
    }
    // NOT #define max(a,b) ( (a) > (b) ? (a) : (b) )

*Инструкция препроцессора #define в С++ в большинстве случаев является излишней (не во всех), потому что С++ предлагает специальные инструкции на замену, например, константы. Иногда лучше даже использовать метод класса для доступа к константе.*

*В примере с макроопределением max может случиться неприятность в следующем коде:*

    int a = 5, b = 0;
    max( ++a, b ); // a увеличиться два раза
    max( ++a, b + 10 ); // а тут всего один

*Таким образом, использование #define разрешается только в тех случаях, когда это крайне необходимо и в языке С++ нет специальной конструкции для этого.*

**[2.2.2] Следует предпочитать библиотеку iostream вместо stdio.**

Пример:

    #include <iostream> // NOT: #include <stdio>

*Функции scanf и printf великолепны, однако у них есть определенные проблемы. Они не осуществляют контроль типов и не расширяемы.*

*В С++ библиотека stdio была заменена более мощной iostream. Поэтому стоит предпочитать ее в большинстве случаев. Однако! Существуют моменты, когда объекты библиотеки stdio работают гораздо быстрее — ради этого правила не следует жертвовать производительностью.*

**[2.2.3] Следует предпочитать приведения типов в стиле С++, вместо С.**

Пример:

    static_cast< double > intValue; // NOT: (double)intValue

*static_cast - аналог обычного приведения типов С. const_cast - преобразование типов для отмены атрибута const. dynamic_cast - безопасный способ downcasting (вернет null если преобразование невозможно). reinterpret_cast нужен для преобразований указателей на функции с изменением сигнатуры (такие действия согласно стандарту С++ не являются переносимыми и зависят от реализации, поэтому использовать это приведение необходимо с осторожностью).*

**[2.2.4] Для операции с указателями следует определить макро NULL (или аналогичный) и использовать его.**

*Следует однако иметь ввиду, что NULL не определяется во многих компиляторах по умолчанию и в своих проектов его надо объявлять явно на уровне foundation.*

**[2.2.5] Следует предпочитать ссылки указателям.**

*Следует использовать только в том случае, если ссылка на объект может не существовать (null). Например, если будет существовать объект «Человек», то родители человека должны быть ссылками (все имеют родителей), в том время как братья и сестры должны быть указателями.*

**[2.2.6] Ключевое слово const должно использоваться везде, где только можно.**

*Ключевое слово const — большой помошник документирования. В частности, методы, которые не изменяют внутреннего состояния класса, ВСЕГДА должны быть объявлены константными.*

**[2.2.7] Не следует использовать передачу объектов «по значению».**

Пример:

    myMethod( const SomeClass& class ); // NOT: myMethod( SomeClass class )

*Существует две основные причины. Первая — быстродействие. Передача «по значению» вызывает создание временного объекта класса с использованием копирующего конструктора, и уничтожение этого объекта при выходе из метода.*

*Вторая причина состоит в том, что объект, переданный «по значению», будет вести себя как базовый класс, и не будет включать информацию наследующего класса.*

**[2.2.8] Следует избегать переменного числа аргументов функции (…).**

*Переменное число аргументов препятствует встроенной строгой проверки типов С++. В большинстве случае следует заменять переменное число аргументов перегрузкой функций и аргументами по умолчанию.*

**[2.2.9] Следует использовать конструкции new и delete вместо malloc, realloc, free.**

*Самая главная проблема функций malloc, free и их вариаций в том, что они ничего не знают о конструкторах и деструкторах.*

*В С++ отсутствует необходимость в старых алгоритмах управления памятью malloc, realloc, free. Кроме того, читаемость кода улучшается при использовании согласованного набора функций управления памятью. Также это делает управление памятью более безопасным.*

*Также следует использовать одинаковые формы new и delete, т.е. new должен соответствовать delete, и new[] должен соответствовать delete[], иначе могут случиться утечки памяти или ее порча. Помните, что при смешивании операторов результат выполнения программы непредсказуем.*

## 2.3. Конструкторы, деструкторы и операторы присваивания ##

**[2.3.1] В классах с динамическим управление памятью всегда определять копирующий конструктор и оператор присваивания.**

*Если конструктор копирования и оператор присваивания не будут явно объявлены, компилятор сделает это автоматически. Если в классе присутствует динамическая работа с памятью, копирующий конструктор и оператор присваивания по умолчанию не будут работать так, как необходимо. Единственное исключение из этого правила состоит в случае, когда следует разделять данные между экземплярами класса, но в таком случае, следует озаботиться, чтобы эти данные не были удалены до окончания использования.*

**[2.3.2] Оператор присваивания всегда должен возвращать ссылку на *this.**

Пример:

    MyClass& MyClass::operator = ( const MyClass& rhs )
    {
        …
        return *this;
    }

*Это необходимо, чтобы типы, объявленные пользователем, вели себя так же, как и встроенные. Для экземпляров приведенного класса возможно и понятно следующее:*

    MyClass a, b, c;
    a = b = c;

*Еще лучше определять оператор присваивания в виде*

    const MyClass& MyClass::operator = ( const MyClass& rhs )

*чтобы избежать недоразумений вида*

    MyClass mc1, mc2, mc3;
    …
    ( mc1 + mc2 ) = mc3; // результату сложения mc1 и mc2 присвоить mc3.

**[2.3.3] Оператор присваивания всегда должен проверять на присваивание самому себе.**

Пример:

    MyClass& MyClass::operator = ( const MyClass& rhs )
    {
        if ( this != &rhs )
        {
            …
        }
    
        return *this;
    }

    MyClass& MyClass::operator = ( const MyClass& rhs )
    {
        if ( *this != rhs )
        {
            …
        }
    
        return *this;
    }

*Какую версию из представленных выбирать — зависит непосредственно от класса. Первая просто проверяет равенство указателей. Этого достаточно в большинстве случаев. Вторая версия проверяет на равенство с использованием оператора != (считается что он определен).*

**[2.3.4] Оператор присваивания класса наследника должен явно производить присваивание через базовый класс.**

Пример:

    Derived& Derived::operator = ( const Derived& rhs )
    {
        if ( this != &rhs )
        {
            Base::operator = ( rhs );
            …
        }
    
        return *this;
    }

*Присваивание базового класса НЕ вызывается автоматически. Некоторые компиляторы могут не позволить существование такой конструкции, если оператор присваивания базового класса генерируется автоматически. В таких случаях следует использовать конструкцию:*

    static_cast<*this> = rhs,

*которая преобразует ссылку в базовый класс, и произведет присваивание через нее.*

**[2.3.5] В конструкторах следует предпочитать инициализацию вместо присваивания.**

*Это нужно дабы не производить двойного присваивания. Это происходит потому, что создание объекта в теории предусматривает два этапа: инициализация членов класса и выполнения тела конструктора. Поэтому если Вы будете использовать присваивание - вы сделаете это дважды.*

*Также следует постулировать и тот факт, что порядок инициализации должен повторять порядок объявления, потому, что компилятор проинициализирует поля класса в том порядке, в котором они были объявлены, а не в том, в котором написана инициализация.*

**[2.3.6] Деструктор должен быть виртуальным только тогда, когда класс содержит виртуальные методы. (Либо этот класс вообще не может быть базовым классом).**

*Когда конструкция delete используется над потомком, базовый класс которого не содержит виртуального деструктора, будет вызван только деструктор базового класса.*

*Другими словами, если класс не содержит виртуальных методов, его не следует использовать как базовый. Это также означает, что не следует объявлять деструктор виртуальным, потому что это ненужно и ведет к увеличению объектов с полностью ненужным vptr и соответствующей vtbl.*

**[2.3.7] Конструкторы объектов принимающих один аргумент должны быть объявлены с ключевым словом explicit, для исключения случаев неявного ненужного приведения.**

*Язык С++ позволяет делать неявные приведения типов, которые могут быть опасны в большинстве случаев. Например,*

    struct vector2f
    {
        …
        vector2f( f32 scalar );
        …
    }

*Теперь мы можем случайно написать так:*

    void doSomething( const vector2f& size ) { … }
    doSomething( 1.0f ); // 1.0f -> vector2f( 1.0f )

*C точки зрения языка проблем нет, однако логически мы видим проблему.*

## 2.4. Операторы ##

**[2.4.1] Оба оператора «!=» и «==» должны определяться всегда, когда нужен хотя бы один из них.**

Пример:

    bool C::operator != ( const C& lhs )
    {
        return !( this == lhs );
    }

*В большинстве случаев достаточно определять оператор «!=» как производный от «==», как показано в примере. Это также будет автоматически сделано компилятором, если один из них будет отсутствовать.*

**[2.4.2] Нельзя перегружать операторы «&&», «||» и «,».**

*Проблема с такими переопределенными операторами может состоят в том, что они будут вести себя не так, как стандартные.*

*С++ производить быстрое вычисление логических выражений, т.е. как только результат становится известным, дальнейшая обработка выражения заканчивается. Нет возможности повторить такое поведение в переопределенных пользователем операторах.*

## 2.5. Наследование ##

**[2.5.1] Отношение «есть разновидность» должно быть смоделировано через наследование, отношение «имеет» - через содержание.**

Пример:

    class B : public A // B “is-a” A
    {
        …
    }

    class B
    {
        …
    
    private:
        A _a; // B “has-a” A
    }
    
**[2.5.2] Следует применять наследование только в том случае, если у наследника изменяется поведение, но не данные**

Пример:

	class Enemy 
	{
		virtual void runAi();
		…
	}
	
	class Boss : public Enemy
	{
		virtual void runAi();
		…
	}
	
Но не

	class Enemy
	{
		…
	public:
		Enemy( ) : _hitpoints(100) { … }
		
	protected:
		int _hitpoints;
	}
	
	class FatEnemy : public Enemy
	{
		…
	public:
		FatEnemy( ) { _hitpoints = 200; }	
	}

**[2.5.3] Не виртуальные методы нельзя переопределять в подклассе.**

*Существует две причины. Если метод должен быть переопределен, то подкласс не должен наследовать его из базового класса, иначе нарушается отношение «является». Во-вторых, существует техническая причина. Не виртуальные методы статически связываются, и ссылки на базовый класс всегда будут вызывать метод базового класса.*

**[2.5.4] Наследованные параметры по умолчанию переопределять нельзя.**

*Относится к предыдущему правилу…*

**[2.5.5] Следует избегать закрытого наследования.**

Пример:

    class C     // NOT: class C: private B
    {           //      {
        …       //          …
    private:    //      }
        B _b;
    }

*В то время как открытое наследование реализует отношение «является», закрытое отношение не реализует ничего, и является всего лишь конструкцией включения кода. Лучше использовать в таких случаях включение.*

*НЕЛЬЗЯ использовать защищенное наследование.*

**[2.5.6] Следует избегать обратного преобразования (downcasting).**

Пример:

    derived = static_cast< DerivedClass* > base;

*Необходимость обратного преобразования свидетельствует о недостатках архитектуры. Хорошо написанный код С++ не должен полагаться на ветвление основанное на типе объекта. («если объект А такого-то типа то сделать это, иначе то»). Лучше использовать виртуальные функции.*

**[2.5.7] Следует избегать использование полиморфизма в массивах.**

Пример:

    class Bst { … }
    class BstBalanced : public Bst { … }

    void printBstArray( ostream& s, const Bst array[], int numElements )
    {
        for ( int i = 0; i < numElements; ++i ) s << array[ i ];
    }

    Bst bstArray[ 10 ];
    printBstArray( cout, bstArray, 10 ); // OK

    BstBalanced bstBalancedArray[ 10 ];
    printBstArray( cout, bstBalancedArray, 10 ); // FAIL!

*Операция array[ i ] вычисляет указатель путем простого арифметического выражения *( array + i ), при этом используется размер смещения sizeof( элемент массива ), что в обоих представленных примерах равно sizeof( Bst ), что естественно неверно во втором случае.*

## 2.6. Исключения ##

**[2.6.1] Если возникает необходимость работы с исключениями, то обрабатывать их по ссылке. Вообще — следует избегать работы с исключениями.**

Пример:

    try
    {
        …
    }
    catch ( Exception& exception )
    {
        …
    }

## 2.7. Разное ##

**[2.7.1] Следует четко проводить различие между методами класса, внешними функциями и дружественными методами.**

*Виртуальные функции должны быть методами класса*

*Операторы operator >> и operator << никогда не должны являться методами класса. Если им необходим доступ к закрытым членам класса, они должны быть дружественными методами.*

*Если методу необходимо преобразование типа на его лево-определенном аргументе, он должен быть внешним. Если в дополнение к этому ему необходим доступ к закрытым полям — он должен быть дружественным.*

*Все остальное должно быть методами класса.*

**[2.7.2] Неявно генерируемые методы, которые не используются, должны быть явно запрещены.**

Пример:

    class C
    {
        …
    
    private:
        C& operator = ( const C& rhs ); // Don't define
    }

*Благодаря такому объявлению метода (как закрытого) и отсутствию определения, попытка доступа к нему будет поймана компилятором.*

*Список методов, который неявно генерируются, если явно не объявлены:*

- Конструктор по-умолчанию ( C::C() )
- Конструктор копирования ( C::C( const C& rhs ) )
- Деструктор ( C::~C() )
- Оператор присваивания ( C& C::operator = ( const C& rhs ) )
- Оператор взятия указателя ( C* C::operator & ( ) )
- Оператор взятия указателя, константный ( const C* C::operator & ( ) const; )

**[2.7.3] Следует предпочитать объекты типа Singleton глобальным переменным.**

Пример:

    class C
    {
    public:
        static const C* getInstance( )
        {
            if ( !_instance ) 
                _instance = new C( );
            
            return _instance;
        }

    private:
        C( );
        static C* _instance; // Defined in the source file
    }

*Подход синглтона решает проблему порядка инициализации глобальных объектов, когда один объект может ссылаться на еще не созданный объект. Вообще, в С++ нет особой необходимости в глобальных переменных.*

**[2.7.4] Функции, которые могут быть реализованы с использованием открытого интерфейса класса, лучше не делать методами.**

*Это может означать то, что логически такая функция является функцией другой сущности. Например, функция поиска пути в графе в общем не является функцией класса графа. Возможно, стоит в таком случае создать специальный класс поиска пути, т.к. в процессе поиска будет собрана дополнительная информация.*

*Например, у вас есть класс "граф". Требуется реализовать функциональность поиска пути. Например, с помощью алгоритмы Дейкстры или A*. Однако, кроме самого пути еще неплохо было бы знать длину этого пути. И кроме того, в ходе алгоритм может сохранять какое-то свое состояние. Таким образом, целесообразнее выделить такой поиск пути не в виде функции, а в виде отдельного класса, например, DejkstraSearch или AStarSearch. Таким образом, найденные показатели и внутренняя структура будут инкапсулированы и сокрыты.*

**[2.7.5] Открытый метод не должен возвращать неконстантную ссылку или указатель на поле класса.**

*В противном случае это нарушает правило инкапсуляции.*

**[2.7.6] Возвращаемый тип функции всегда должен быть явно объявлен.**

Пример:

    int function( )     // NOT: function( )
    {                   //      {
        …               //          …
    }                   //      }

*Функции, для которых явно не объявлен возвращаемый аргумент, неявно получают int в качестве возвращаемого значения. Функции не должны основываться на этом факте.*

**[2.7.7] При определении константы на указатель в заголовочном файле, необходимо сам указатель также объявлять как константный:**

    const char* const someString = “This is a string”;

*Это необходимо ввиду того, что при определении константы в заголовочном файле к ней могут иметь доступ множество различных исходных файлов. Также при объявлении констант в рамках класса стоит их делать статическим (static) членами, как:*

    *.h
    
    class SomeClass
    {
    private:
        static const int SOME_CONSTANT = 5;
        int _someArray[ SOME_CONSTANT ];
        
        …
    };

    *.cpp
    
    const int SomeClass::SOME_CONSTANT;

**[2.7.8] Избегать перегрузки по численному типу и указателю.**

    void f( int x );
    void f( string* str );
    f( 0 ); // Что будет вызвано? f(int) или f(string*).

*В примере 0 - это литерная целая константа, т.е. всегда будет вызвана функция f(int). Чтобы не создавать проблемных ситуаций такая перегрузка запрещена, потому что даже если найти способ “правильного” определения NULL, заставить программистов пользоваться им Вы не в силах.*

**[2.7.9] Следует избегать создание потенциальной неоднозначности:**

    class B;
    
    class A
    {
    public:
        A( const B& );
    };

    class B
    {
    public:
        operator A( ) const;
    };

*Ввиду того, что имея экземпляр одного из классов всегда можно получить экземпляр другого, то может возникнуть потенциальная неоднозначность, отследить которую невозможно, потому что формально код абсолютно корректен. Поэтому такая особенность может стать “миной замедленного действия”.*

**[2.7.10] Различайте наследование и шаблоны**

*При выборе того, будет ли класс реализовываться посредством шаблонов или наследования следует руководствоваться следующими соображениями:*

1. шаблоны должны использоваться для генерации семейств классов, тип объектов которых не влияет на поведение функций этих классов;
2. наследование должно использоваться для создания семейств классов, тип объектов которых влияет на поведение функций создаваемых классов.

**[2.7.11] Не стоит использовать #include, когда возможно предварительное объявление**

*Когда вы подключаете заголовочный файл - вы создаете зависимость, которая приводит к тому, что код перекомпилируется при изменении этого заголовочного файла. При построении сложных цепочек зависимостей, количество кода для перекомпиляции сильно возрастает. Поэтому, мы рекомендуем минимизировать количество директив #include, особенно в заголовочных файлах.*

## Список использованных источников ##

- Нотация iLLi выпуска от 7 апреля 2010 года.
- Effective C++ Second Edition, Scott Meyers – Addison-Wesley 1998
- More effective C++, Scott Meyers – Addison-Wesley, 1996
- How Non-Member Functions Improves Incapsulation, Scott Meyers – C/C++ Users Journal, February, 2000. http://www.cuj.com/archive/1802/feature.html
- Programming in C++, Rules and Recommendations, M. Henricson / E. Nyquist, 1992.
- Crytek coding rules.
- International Strandard for Information Systems - Programming Language C++.

## Специальные благодарности ##

- Малых Денису Александровичу и Дейнеге Василию Михайловичу за многолетнюю поддержку нотации iLLi.
- Компании Geotechnical Software Services за вдохновление на написание этого документа, полученное при прочтении их рекомендаций кодированния.