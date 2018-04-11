---
layout: post
title:  "Паттерны из книги Confident Code by Avgi Grimm"
date:   2018-04-10 00:05:41 +0900
category: ruby
---

Основная концепция книги сводится к тому, чтобы все процессы сводились к 4 шагам:
- Collecting input
- Performing work
- Delivering output
- Handling failures

![Процесс](http://dl3.joxi.net/drive/2018/04/10/0018/2910/1186654/54/62df256b6b.jpg)

Далее нам необходимо всегда делить виртульно наш код на сообщения и получателя.
Выглядит это так:

![message_role](http://dl3.joxi.net/drive/2018/04/10/0018/2910/1186654/54/ff7d5be6a0.jpg)

В книге много примеров, которые по мнению автора, должны работать с разными аргументами для методов и классов.
Автор так же намекает, что обычно это происходит с API.
Если мы работаем с внутренней реализацией, то по-моему мнению это чаще всего лишнее и для корректной работы достаточно поддерживать только один ожидаемый тип аргумента.

Итак по порядку.

## Collecting Input

- # Introduction to collecting input
  Разделять методы по их основным предназначением.
  Одна задача на класс/метод дает нам возможность уменьшить связность между кодовой базой и уменьшить потенциальные баги.
  Создавать классы и константы, которые можно дальше использовать в других классах/методах. Если что-то нужно поменять, то поменяется в одном месте, а не во всей кодовой базе.

- # Use built-in conversion protocols
Встроенные протоколы преобразования полезны когда необходимо дать объекту поведение схожее с другим типом.
Это делается путем неявного вызова встроенных методов руби

Таблица явных и не явных методов:

![table](http://dl3.joxi.net/drive/2018/04/11/0018/2910/1186654/54/f709963af5.jpg)
![table](http://dl3.joxi.net/drive/2018/04/11/0018/2910/1186654/54/787932ff80.jpg)

Т.к. мы работаем с объектно-ориентированным языком, то мы обычно оборачиваем все в объекты.

Допустим если у нас есть массив

{% highlight ruby %}

  winners = [
    "Homestar",
    "King of Town",
    "Marzipan",
    "Strongbad"
  ]
{% endhighlight %}

И нам нужна функция, которая бы выводила из массива призёров в каком-то виде.
Мы можем воспользоваться Struct.

{% highlight ruby %}
  Place = Struct.new(:index, :name, :prize) do
    def to_int
      index
    end
  end
{% endhighlight %}

В структуре мы определили не явный метод to_int. Этот трюк позволит делать вот так:
{% highlight ruby %}

  first = Place.new(0, :first, :car)
  winners[first]
  #=> 'Homestar'
{% endhighlight %}

Или вот еще:

{% highlight ruby %}

  class EmacsConfigFile
    def initialize
      @filename = "#{ENV['HOME']}/.emacs"
    end

    def to_path
      @filename
    end
  end

  emacs_config = EmacsConfigFile.new
  File.open(emacs_config).lines.count
{% endhighlight %}

Еще пример:

{% highlight ruby %}
 'Сейчас' + Time.now
  #=>can't convert Time into String (TypeError)
{% endhighlight %}

Все дело в том, что у времени нет не явного метода to_str.
А вот если мы будем использовать интерполяцию, то будем вызван явный метод to_s

{% highlight ruby %}
 "Сейчас #{Time.now}"
  #=>Сейчас 2018-04-11 10:09:17 +0900
{% endhighlight %}

В каких случаях нам необходимо использовать явные или не явные методы?

Явные методы даны программистам, чтобы человек сам принимал решение когда необходимо принудительно привести к определенному типу.

Бывает методы принимают аргументы, но аргумент может оказаться nil.

Приведу пример:

{% highlight ruby %}
 def set_speed(rpm)
   value = rpm.to_i
   puts "current speed is #{value}"
 end
{% endhighlight %}

Что будет если аргументом будет nil?


{% highlight ruby %}
 nil.to_i
 #=> 0
{% endhighlight %}

В этом случае у нас будет вывод текущей скорости возможно не тот, что мы ждем.

Чтобы у нас сразу произошла ошибка, можно сделать несколькими способами. Например добавить условие с проверкой.
Но книга 'уверенный код' показывает, что мы можем это сделать более удобно.

Изменим наш пример:

{% highlight ruby %}
 def set_speed(rpm)
   value = rpm.to_int
   puts "current speed is #{value}"
 end

 set_speed(nil)
 #=> undefined method `to_int' for nil:NilClass (NoMethodError)
{% endhighlight %}

Вот оно! Стразу ошибка. Мы сможем быстро отреагировать и понять где ошибка по stacktrace.

Под словом confident code подразумевается, что наш код будет иметь как можно меньше проверок, больше ясности кода, как будто мы читаем книгу.
Далее еще будут примеры, где мы все-таки будем использовать raise и другие техники, но это все-лишь для того, чтобы опять же упростить и свести все проверки к минимуму. Чтобы наша программа сразу же нам говорила, где проблема, а не скрывала ее до тех пор, пока где-то глубоко не появится ошибка, которую будет уже в разы сложнее диагностировать.

- # Define your own conversion protocols
Бывают ситуации, когда необходимо сделать свой протокол, т.к. в руби отсутствуют такие.
Например наш метод ожидает координаты. Обычно это X,Y.


{% highlight ruby %}

  def draw_line(start, endpoint)
    start = start.to_coords if start.respond_to?(:to_coords)
    start = start.to_ary
  end

  class Point
    attr_reader :x, :y, :name

    def initialize(x, y, name=nil)
      @x, @y, @name = x, y, name
    end

    def to_coords
      [x,y]
    end
  end

  start = Point.new(23, 37)
  endpoint = [45,89]
  draw_line(start, endpoint)
{% endhighlight %}

- # Define conversions to user-defined types
Добавляем в метод принудительное ожидание объекта с определенным интерфейсом.
Это позволит сторонним пользователям API использовать свои собственные классы, которые образают свою логику, но всегда возвращают какой-то результат в ожидаемом интерфейсе(Вообще по идее мы могли бы просто передавать в метод уже готовые значения. Но судя по всему, концепция объектно-ориентированного программирования сводится к тому, что все должно быть объектом. Если вы посмотрите в ROR, то там например почти всегда во всех переменных хранится какой-то объект. Наприме объект request/response в себе содержит переменные, значения которых являются объект, к которому можно обратиться и использовать методы объекта).

Пример:

{% highlight ruby %}

  def report_altitude_change(current_altitude, previous_altitude)
    change = current_altitude.to_meters - previous_altitude.to_meters
  end

  class Meters
    ...
    def to_meters
     self
    end

    def -(other)
      self.class.new(value - other.to_meters.value)
    end
  end

  class Feet
    ...
    def to_meters
      Meters.new((value * 0.3048).round)
    end
  end

  report_altitude_change(Meters.new(100), Feet.new(30))
{% endhighlight %}

- # Use built-in conversion functions
Используйте встроенные функции преобразования.
В руби мы можем на любом объекте использовать методы вида to_i, to_s, to_h.
Но эти методы могут вернуть не то, что мы ожидаем.
В Kerner есть функции :Array(), Float(), String(), Integer(), Rational(), Complex() etc.
Они позволяют гарантированно вернуть то, что нам нужно. В противном случае будет ошибка.

{% highlight ruby %}

  Integer(10) # => 10
  Integer(10.1) # => 10
  Integer("0x10") # => 16
  Integer("010") # => 8
  Integer("0b10") # => 2
  # Time defines #to_i
  Integer(Time.now) # => 1341469768

  "ham sandwich".to_i # => 0
  Integer("ham sandwich") # =>
  # ~> -:2:in `Integer': invalid value for Integer(): "ham sandwich" (ArgumentError)


  Array("foo") # => ["foo"]
  Array([1,2,3]) # => [1, 2, 3]
  Array([]) # => []
  Array(nil) # => []
  Array({:a => 1, :b => 2}) # => [[:a, 1], [:b, 2]]
  Array(1..5) # => [1, 2, 3, 4, 5]

{% endhighlight %}

- # Write total functions
Чтобы уменьшить кол-во ошибок если метод вернет nil или пустую строку, можно воспользоваться методами массива, чтобы всегда возвращать массив.

- # Replace "string typing" with classes

Заменяйте строковые значения на классы. Это позволит избежать внутренних ошибок в программе, т.к. неверное имя класса приведет к ошибке.

Пример плохого класса:
{% highlight ruby %}

  class TrafficLight
    def change_to(state)
      @state = state
    end

    def signal
      case @state
      when "stop" then turn_on_lamp(:red)
      when "caution"
        turn_on_lamp(:yellow)
        ring_warning_bell
      when "proceed" then turn_on_lamp(:green)
      end
    end

    def next_state
      case @state
      when "stop" then "proceed"
      when "caution" then "stop"
      when "proceed" then "caution"
      end
    end

    def turn_on_lamp(color)
      puts "Turning on #{color} lamp"
    end

    def ring_warning_bell
      puts "Ring ring ring!"
    end
  end

  light = TrafficLight.new
  light.change_to("PROCEED") # oops, uppercase
  light.signal
  puts "Next state: #{light.next_state.inspect}"
  light.change_to(:stop) # oops, symbol
  light.signal
  puts "Next state: #{light.next_state.inspect}"
  Next state: nil
  Next state: nil

{% endhighlight %}

Попробуем улучшить:

{% highlight ruby %}

    def next_state
      @state.next_state
    end

    def State(state)
      case state
      when State then state
      else self.class.const_get(state.to_s.capitalize).new
      end
    end

    class Stop < State
      def color; 'red'; end
      def next_state; Proceed.new; end
    end

    class Caution < State
      def color; 'yellow'; end
      def next_state; Stop.new; end

      def signal(traffic_light)
        super
        traffic_light.ring_warning_bell
      end
    end

    class Proceed < State
      def color; 'green'; end
      def next_state; Caution.new; end
    end

    light = TrafficLight.new
    light.change_to(:caution)
    light.signal
    puts "Next state is: #{light.next_state}"
    Turning on yellow lamp
    Ring ring ring!
    Next state is: stop

{% endhighlight %}

- # Wrap collaborators in Adapters

Паттерн адаптер позволяет для какого-то объекта реализовать интеферс, который ожидает другой класс для корректной работы.

Допустим у нас есть класс логгер

{% highlight ruby %}

  class BenchmarkedLogger
    def initialize(sink=$stdout)
      @sink = sink
    end

    def info(message)
      start_time = Time.now
      yield
      duration = start_time - Time.now
      @sink << ("[%1.3f] %s\n" % [duration, message])
    end
  end

{% endhighlight %}

Этот код работает прекрасно с объектами, которые поддерживают метод <<. Например $stdout

Но, что если у нас будет объект у которого нет метода << ?
Для этого напишем адаптер, который позволит нам работать с текущей реализацией без проблем.

{% highlight ruby %}

  class BenchmarkedLogger
    class IrcBotSink
      def initialize(bot)
        @bot = bot
      end

      def <<(message)
        @bot.handlers.dispatch(:log_info, nil, message)
      end
    end

    def initialize(sink)
      @sink = case sink
      when Cinch::Bot then IrcBotSink.new(sink)
      else sink
      end
    end
  end

{% endhighlight %}

Таким образом у нас все инкапсулировано в отдельный класс с ожидаемым интерфейсом. Концепция уверенного кода подразумевает все инкапсулировать, чтобы у нас в методах были только ясные и понятный инструкции.

- # Use transparent adapters to gradually introduce abstraction

Прозрачный адаптер - это просто по сути объект, который делегирует все методы, которые отстутвуют в самом адаптере объекту, на котором адаптер был вызван.

Из предыдушего примера с Logger мы можем переделать его так:

{% highlight ruby %}

  require 'delegate'
  class BenchmarkedLogger
    class IrcBotSink < DelegateClass(Cinch::Bot)
      def <<(message)
        handlers.dispatch(:log_info, nil, message)
      end
    end

    def initialize(sink)
      @sink = case sink
      when Cinch::Bot then IrcBotSink.new(sink)
      else sink
      end
    end
  end
{% endhighlight %}

- # Reject unworkable values with preconditions

Смысл этого паттерна вызывать сразу ошибку, если класс ожидает аргументы, которые должны быть 100% обязательными.
Если эти аргументы используют многие методы, которые могут привести к ошибке, вместо того, чтобы делать кучу проверок, мы можем сделать проверку сразу при инициализации класса.

{% highlight ruby %}

  class Employee
    attr_accessor :name
    attr_reader :hire_date

    def initialize(name, hire_date)
      @name = name
      self.hire_date = hire_date
    end

    def hire_date=(new_hire_date)
      raise TypeError, "Invalid hire date" unless
      new_hire_date.is_a?(Date)
      @hire_date = new_hire_date
    end

    def due_for_tie_pin?
      ((Date.today - hire_date) / 365).to_i >= 10
    end

    def covered_by_pension_plan?
      hire_date.year < 2000
    end

    def bio
      "#{name} has been a Yoyodyne employee since #{hire_date.year}"
    end
  end
{% endhighlight %}


- # Use #fetch to assert the presence of Hash keys
Используйте метод fetch для получения ключа в хэше.

{% highlight ruby %}
  attributes.fetch(:login)
  #<KeyError: key not found: :password>

  attributes.fetch(:password) do
    raise KeyError, "Password (or false) must be supplied"
  end
{% endhighlight %}

- # Use #fetch for defaults

fetch можно так же использовать для установки значения по-умолчанию

{% highlight ruby %}
  attributes.fetch(:logger) { Logger.new($stdout) }
  attributes.fetch(:logger, Logger.new($stdout) )
{% endhighlight %}


- # Handle special cases with a Guard Clause

Позволяет сделать return по условию и не делаеть лишних if else

{% highlight ruby %}
  def current_logged_users
    return o if logged_users.nil?

    logged_users.size
  end
{% endhighlight %}


- Represent special cases as objects aka NullObject

Чтобы не делать лишние условия иногда удобно воспользоваться этим паттерном.
Пример:

{% highlight ruby %}
  def current_user
    logged_user || NullUser.new
  end

  class NullUser
    def name
      :guest
    end
  end

  "Hello #{current_user.name}"
{% endhighlight %}


- # Yield a parameter builder object
Это просто передача в блок аргумента, с которым можно работать. Это удобнее выглядит.

{% highlight ruby %}
  def home_builder
    buulder = OtherBuilder.new
    yield(buulder) if block_given?
    ...
  end


  home_builder do |builder|
    builder.name = :name
    builder.other = :other
  end
{% endhighlight %}


- # Receive policies instead of data

При работе с возможными исключениями бывает необходимо по-разному отработать ошибку.

Для этого мы можем использовать несколько путей, но более гибкий является передача управления в блок или использования политик.
Вот как это выглядит:

{% highlight ruby %}

  def delete_files(files)
    files.each do |file|
      begin
        File.delete(file)
      rescue => error
        if block_given? then yield(file, error) else raise end
      end
    end
  end

  delete_files(files) do |file, error|
    send_email(file, error)
  end

  delete_files(files) do |file, error|
    puts "can't delete #{file} with #{error}"
  end

{% endhighlight %}

{% highlight ruby %}

  def delete_files(files, options={})
    error_policy =
    options.fetch(:on_error) { ->(file, error) { raise error } }
    symlink_policy =
    options.fetch(:on_symlink) { ->(file) { File.delete(file) } }
    files.each do |file|
      begin
        if File.symlink?(file)
          symlink_policy.call(file)
        else
          File.delete(file)
        end
      rescue => error
        error_policy.call(file, error)
      end
    end
  end

{% endhighlight %}

- # Call back instead of returning

Этот паттерн включает в себя подключение политик и паттерна command-query separation (CQS).
CQS подразумевает, что метод возвращает либо команду, либо значение, но не оба.

{% highlight ruby %}

  def import_purchase(date, title, user_email, &import_callback)
    user = User.find_by_email(user_email)
    unless user.purchased_titles.include?(title)
      purchase = user.purchases.create(title: title, purchased_at: date)
      import_callback.call(user, purchase)
    end
  end

  import_purchase(date, title, user_email) do |user, purchase|
    send_book_invitation_email(user.email, purchase.title)
  end
{% endhighlight %}

Очень похоже на yield, за тем исключеним, что мы можем сделать много блоков(политик) и передавать методу в каждой ситуации свою политику.

- # Yield a status object

Мой любимый паттерн. Позволяет передавать статус результат выполнения функции и выполянть пользовательские действия дальше.

Предыдущий пример был хороший, но у него нет ничего чтобы можно было отреагировать на ошибку или иные результаты выполнения метода.
Мы могли бы использовать хэш и в него передавать полики, но тут немного удобнее.

{% highlight ruby %}

  class ImportStatus
    def self.success() new(:success) end
    def self.redundant() new(:redundant) end
    def self.failed(error) new(:failed, error) end

    attr_reader :error

    def initialize(status, error=nil)
      @status = status
      @error = error
    end

    def on_success
      yield if @status == :success
    end

    def on_redundant
      yield if @status == :redundant
    end

    def on_failed
      yield(error) if @status == :failed
    end
  end

  def import_purchase(date, title, user_email)
    user = User.find_by_email(user_email)
    if user.purchased_titles.include?(title)
      yield ImportStatus.redundant
    else
      purchase = user.purchases.create(title: title, purchased_at:
    date)
      yield ImportStatus.success
    end
    rescue => error
      yield ImportStatus.failed(error)
    end
  end

  import_purchase(date, title, user_email) do |result|
    result.on_success do
      send_book_invitation_email(user_email, title)
    end

    result.on_redundant do
      logger.info "Skipped #{title} for #{user_email}"
    end

    result.on_error do |error|
      logger.error "Error importing #{title} for #{user_email}: #{error}"
    end
  end
{% endhighlight %}
