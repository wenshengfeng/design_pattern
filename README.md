# desig_pattern (设计模式 李建忠)
---
- **`面向对象设计原则`**
  1.单一职责原则（SRP）
  2.开闭原则（OCP）
  3.里氏代换原则（LSP）
  4.依赖倒置原则（DIP）
  5.接口隔离原则（ISP）
  6.合成复用原则（CRP）
  7.迪米特法则（LoD）
- **`泛型编程`** 与 **`代码重构`**
  **推荐书籍** `重构与模式` `重构-改善既有代码的设计`
  **重构关键技法**
  - 静态->动态
  - 早绑定->晚绑定
  - 继承->组合
  - 编译时依赖->运行时依赖
  - 紧耦合->松耦合
---
### 典型模式
- **模板方法(Template Method)**
  模板方法模式是一种非常基础性的设计模式，在面向对象系统中有着大量的应用。它用最简单的机制（虚函数的多态）为程序应用框架提供了灵活的扩展点，是代码复用方面的基本实现机构。
  ```cpp
    class Libray{
        public:
        void run(){ //稳定
            if(init())
            {
                go_run();
            }
        }
        virtual ~Libray(){};

        protected:
        void go_run(){} // 稳定
        virtual bool init() = 0 ; // 变化

    };

    class Appliation : public Libray{
        public:
        bool init(){

        }
        ~Appliation(){}
    };

    int main()
    {
        Libray *lib = new Appliation();
        lib->run();
        delete lib;
        return 0;
    }
  ```
 - **策略模式(Strategy Pattern)**
    在软件构建过程中，某些对象使用的算法可能多种多样，经常改变，如果将这些算法都编码到对象中，将会使对象变得异常复杂；而且有时候支持不使用的算法也是一个性能负担。本模式是解决将算法与对象本身进行解耦，从而避免上述问题。(消除if判断问题)
    ```cpp
    #include <iostream>

    class Strategy{
        public:
        virtual double doOperation(double num1, double num2) = 0;
        virtual ~Strategy(){}
    };

    class OperationAdd:public Strategy{
        public:
        double doOperation(double num1, double num2) override{
            return num1 + num2;
        }
        ~OperationAdd(){}
    };

    class OperationSub: public Strategy{
        public:
        double doOperation(double num1, double num2) override{
            return num1 - num2;
        }
        ~OperationSub(){}
    };

    class Context{
        private:
        Strategy* strategy;
        public:
        Context( Strategy* strategy){
            this->strategy = strategy;
        }

        double executeStrategy(double num1 , double num2){
            return strategy->doOperation(num1,num2);
        }

        ~Context(){
            delete strategy;
        }
    };

    int main()
    {
        Context* ctx = new Context(new OperationSub());
        double rlt = ctx->executeStrategy(6.1, 3.0);
        std::cout<<"rlt is :" << rlt << std::endl;

        return 0;
    }
    ```
- **观察者模式(Observer)**
  观察者模式使用三个类 Subject、Observer 和 Client。Subject 对象带有绑定观察者到 Client 对象和从 Client 对象解绑观察者的方法。我们创建 Subject 类、Observer 抽象类和扩展了抽象类 Observer 的实体类。
  ObserverPatternDemo，我们的演示类使用 Subject 和实体类对象来演示观察者模式。
  ```cpp
    #include <string>
    #include <list>
    #include <iostream>

    class Observer
    {
    public:
        virtual ~Observer(){std::cout << "delete Observer" << std::endl;}
        virtual void update(unsigned int value) = 0;
    };

    class ObserverList
    {
    private:
        std::list<Observer*> m_observerList;
    public:
        void add(Observer *observer) //添加观察者
        {
            m_observerList.push_back(observer);
        }
        void remove() //删除观察者
        {
            m_observerList.pop_back();
        }
        void notifyAllObservers(unsigned int value) //变化通知
        {
            for (auto observr:  m_observerList)
            {
                observr->update(value);
            }
        }
    };
    class FileSpilter
    {
    private:
        std::string m_filePath;
        unsigned int m_fileNumber;
        ObserverList* m_observerList;
    public:
        FileSpilter(std::string& path, unsigned int fileNumber, ObserverList* observer)
        :m_filePath(path),
        m_fileNumber(fileNumber),
        m_observerList(observer)
        {}
        ~FileSpilter(){}
        void spilt(){
            //处理业务
            //.....

            //状态通知
            int values = m_fileNumber;
            m_observerList->notifyAllObservers(values);
        }
    };

    class MyObserver:public Observer{
        public:
        void update(unsigned int value) override {
            std::cout << "update MyObserver values is :" << value << std::endl;
        }
        ~MyObserver(){ std::cout<< "delete MyObserver"<<std::endl;}
    };

    class MyObserver1:public Observer{
        public:
        void update(unsigned int value) override {
            std::cout << "update MyObserver1 values is :" << value << std::endl;
        }
        ~MyObserver1(){ std::cout<< "delete MyObserver1"<<std::endl;}
    };

    int main()
    {
        ObserverList* oblist = new ObserverList();
        oblist->add(new MyObserver()); //添加观察者
        oblist->add(new MyObserver1());

        std::string path = "/home/a.txt";
        FileSpilter* spilter = new FileSpilter(path, 23, oblist);
        spilter->spilt();

        //删除观察者
        oblist->remove();

        return 0;
    }
  ```
### "单一职责模式"
在软件组件的设计中，如果责任划分不清晰，使用继承得到的结果往往是随着需求的变化，子类急剧膨胀，同时充斥着重复代码，这个时候的关键是划分责任。
- **装饰器模式（Decorator）**
  装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。
  这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。(**组合与继承一起使用**)
  ```cpp
    class Stream{
        public:
        virtual char Read(unsigned int number) = 0;
        virtual void Seek(unsigned int pos) = 0;
        virtual void Write(char data) = 0;
        virtual ~Stream(){}
    };

    class FileStream:public Stream{
    public:
        char Read(unsigned int number) override{
            // 文件具体操作
            return 'a';
        }
        void Seek(unsigned int pos) override{

        }
        void Write(char data) override{

        }
    };

    class NetWorkStream:public Stream{
    public:
        char Read(unsigned int number) override{
            //网络流操作
            return 'a';
        }
        void Seek(unsigned int pos) override{

        }
        void Write(char data) override{

        }
    };


    //加密流操作
    class CryptoStream:public Stream{
        Stream * stream; //多态对象 可以是文件流 网络流
    public:
        CryptoStream(Stream *stm):stream(stm){}
        char Read(unsigned int number) override{
            //额外加密操作
            
        return stream->Read(number);
        }
        void Seek(unsigned int pos) override{
            //额外加密操作

            stream->Seek(pos);
        }
        void Write(char data) override{
            //额外加密操作

            stream->Write(data);
        }
    };


    int main()
    {
        FileStream * fileStream = new FileStream();
        CryptoStream * cryoto = new CryptoStream(fileStream);
        return 0;
    }
  ```
  **完整版**
  ```cpp
    class Stream{
        public:
        virtual char Read(unsigned int number) = 0;
        virtual void Seek(unsigned int pos) = 0;
        virtual void Write(char data) = 0;
        virtual ~Stream(){}
    };

    class FileStream:public Stream{
    public:
        char Read(unsigned int number) override{
            // 文件具体操作
            return 'a';
        }
        void Seek(unsigned int pos) override{

        }
        void Write(char data) override{

        }
    };

    class NetWorkStream:public Stream{
    public:
        char Read(unsigned int number) override{
            //网络流操作
            return 'a';
        }
        void Seek(unsigned int pos) override{

        }
        void Write(char data) override{

        }
    };

    class DecoatorStream:public Stream{
    protected:
        Stream * stream;
    public:
        DecoatorStream(Stream *stm):stream(stm){}
        char Read(unsigned int number) override{
            return stream->Read(number);
        }
        void Seek(unsigned int pos) override{
            stream->Seek(pos);
        }
        void Write(char data) override{
            stream->Write(data);
        }
    };

    //加密流操作
    class CryptoStream:public Stream{
        DecoatorStream * stream; //多态对象 可以是文件流 网络流
    public:
        CryptoStream(DecoatorStream *stm):stream(stm){}
        char Read(unsigned int number) override{
            //额外加密操作
            
        return stream->Read(number);
        }
        void Seek(unsigned int pos) override{
            //额外加密操作

            stream->Seek(pos);
        }
        void Write(char data) override{
            //额外加密操作

            stream->Write(data);
        }
    };


    int main()
    {
        FileStream * fileStream = new FileStream();
        DecoatorStream *decorator = new DecoatorStream(fileStream);
        CryptoStream * cryoto = new CryptoStream(decorator);
        return 0;
    }
  ```
- **桥模式(Bridge)**
  桥接（Bridge）是用于把抽象化与实现化解耦，使得二者可以独立变化。这种类型的设计模式属于结构型模式，它通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦。这种模式涉及到一个作为桥接的接口，使得实体类的功能独立于接口实现类。这两种类型的类可被结构化改变而互不影响。我们通过下面的实例来演示桥接模式（Bridge Pattern）的用法。其中，可以使用相同的抽象类方法但是不同的桥接实现类，来画出不同颜色的圆。
  ```cpp
    #include <iostream>
    #include <stdexcept>

    class DrawAPI{
    public:
        virtual void drawCircle(int radius, int x, int y) = 0 ;
        virtual ~DrawAPI(){}
    };

    class RedCircle : public DrawAPI{
    public:
        void drawCircle(int radius, int x, int y) override {
            std::cout << "Drawing Circle [ color : red, radius : " << radius << " x:" << x << " y: " << y << std::endl;
        }
        ~RedCircle(){std::cout << "destructor RedCircle" << std::endl;}
        
    };


    class GreenCircle : public DrawAPI{
    public:
        void drawCircle(int radius, int x, int y) override {
            std::cout << "Drawing Circle [ color : green, radius : " << radius << " x:" << x << " y: " << y << std::endl;
        }
        ~GreenCircle(){std::cout << "destructor GreenCircle" << std::endl;}
    };

    class Shape {
    protected:
        DrawAPI* drawApi;
    public:
        virtual void draw() = 0;
        virtual ~Shape(){delete drawApi; std::cout << "destructor Shape:drawApi" << std::endl;}
    };

    class Cirecle :public Shape{
        public:
        Cirecle(int x, int y, int radius, DrawAPI* api)
        {
            drawApi= api;
            this->radius = radius;
            this->x = x;
            this->y = y;
        }
        ~Cirecle(){}
        void draw() override {
            drawApi->drawCircle(radius, x, y);
        }
        private:
        int x, y ,radius;
    };

    auto main () -> int {
        Shape*  redCircle = new Cirecle(100,100, 10, new RedCircle());
        redCircle->draw();
        delete redCircle;

        Shape* greenCircle = new Cirecle(101,101, 11, new GreenCircle());
        greenCircle->draw();
        delete greenCircle;

        return  0;
    }

  ```
- **工厂方法模式(Factory)**
  工厂模式（Factory Pattern）是 `C++` 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。(*作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。*)**对象构造方法 必须一致，不然不能使用工厂方法**
  ```cpp
    #include <type_traits>
    #include <iostream>

    class ISpilt{
        public:
        virtual void spilt() = 0;
        virtual ~ISpilt(){}
    };

    class TxtSpilter : public ISpilt{
        public:
        void spilt() override{
            /// 具体实现
            std::cout << "TxtSpilter spilt txt" << std::endl;
        }
    };

    class PhotoSpilter: public ISpilt{
        public:
        void spilt() override{
            //具体实现   
            std::cout << "PhotoSpilter spilt " << std::endl;   
        }
    };

    class FatorySpilter{
        public:
        virtual ISpilt* CreateFatorySpilter() = 0;
        virtual ~FatorySpilter(){};
    };

    class TextFatorySpilter: public FatorySpilter{
        public:
        ISpilt * CreateFatorySpilter() override{
            return new TxtSpilter();
        }
    };

    class PhotoFatorySpilter: public FatorySpilter{
        public:
        ISpilt * CreateFatorySpilter() override{
            return new PhotoSpilter();
        }
    };

    auto main () -> int {
        FatorySpilter* fatory = new TextFatorySpilter(); // 通过外界传入具体的实体
        ISpilt* spilt= fatory->CreateFatorySpilter();
        spilt->spilt();

        delete fatory;
        delete spilt;
        
        return 0;
    }
  ```
- **抽象工厂方法(Abstract Factory)**
  抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
  在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。
  ```cpp
    #include <iostream>
    #include <string_view>

    //DB连接
    class IDBConnection{
        public:
        virtual bool connect_db(std::string_view db_msg) = 0;
        virtual ~IDBConnection(){}
    };

    // DB SQL 执行
    class IDBCommand{
        public:
        virtual bool command_exector(std::string_view cmd) = 0;
        virtual ~IDBCommand(){}
    };

    //DB结果获取
    class IDBReader{
        public:
        virtual bool db_read() = 0;
        virtual ~IDBReader(){}
    };

    // MySQL的db conection 实例
    class MySQLDBConnection: public IDBConnection{
        public:
        bool connect_db(std::string_view db_msg) override{
            //连接MySQL 数据库
            // .....
            return true;
        }
    };
    // Oarcle 的 db 实例
    class OarcleDBConnection: public IDBConnection{
        public:
        bool connect_db(std::string_view db_msg) override{
            //连接Oarcle 数据库
            // .....
            return true;
        }
    };

    // MySQL 的 Command 实例
    class MySqlDBCommand:public IDBCommand{
        public:
        bool command_exector(std::string_view cmd) override{
            // 执行 Mysql SQL 
            // ...
            return true;
        }
    };
    // Oarcle  的 Command 实例
    class OarcleDBCommand:public IDBCommand{
        public:
        bool command_exector(std::string_view cmd) override{
            // 执行 Oarcle SQL 
            // ...
            return true;
        }
    };

    // Mysql 的 db Reader 实例
    class MySqlDBReader: public IDBReader{
        public: 
        bool db_read() override{
            // 执行 mysql 读取操作
            // ...
            return true;
        }
    };
    // Oarcle 的 db Reader 实例
    class OarcleDBReader: public IDBReader{
        public: 
        bool db_read() override{
            // 执行 Oarcle 读取操作
            // ...
            return true;
        }
    };

    // IDBConnection IDBCommand IDBReader 有关联性 必须为同种数据库的操作类型
    // 所以 将提供生成这三个关联抽象的工厂
    class IDBAbstractFactory{
        public:
        virtual IDBConnection* CreateDBConnection() = 0;
        virtual IDBCommand* CreateDBCommand() = 0;
        virtual IDBReader* CreateDBReader() = 0;
        virtual ~IDBAbstractFactory(){}
    };

    // 抽象工厂实例 MySqlFactory
    class MySqlDBFactory: public IDBAbstractFactory{
        public:
        IDBConnection * CreateDBConnection() override{
            return new MySQLDBConnection();
        }
        IDBCommand * CreateDBCommand() override{
            return new MySqlDBCommand();
        }
        IDBReader * CreateDBReader() override{
            return new MySqlDBReader();
        }
    };

    // 抽象工厂实例 OarcleFactory
    class OarcleDBFactory: public IDBAbstractFactory{
        public:
        IDBConnection * CreateDBConnection() override{
            return new OarcleDBConnection();
        }
        IDBCommand * CreateDBCommand() override{
            return new OarcleDBCommand();
        }
        IDBReader * CreateDBReader() override{
            return new OarcleDBReader();
        }
    };


    int main()
    {
        IDBAbstractFactory* factoty = new MySqlDBFactory(); //具体是Mysql 还是 Oracle 从外界传入
        
        IDBConnection* connection =factoty->CreateDBConnection();
        connection->connect_db("db ip prot ");
        IDBCommand* cmd = factoty->CreateDBCommand();
        cmd->command_exector("select * from a");
        IDBReader* reader = factoty->CreateDBReader();
        reader->db_read();

        return 0;
    }
  ```
- **原型模式(Prototype)**
  原型模式（Prototype Pattern）是用于创建重复的对象，同时又能保证性能。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
  这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。
  ```cpp
    class ISpilter{
        public:
        virtual ISpilter* clone() = 0;
        virtual void spilt() = 0;
        virtual ~ISpilter(){}
    };

    // 
    class TextSpilter:public ISpilter{
        public:
        ISpilter * clone() override{
            return new TextSpilter(*this); // 深克隆 以拷贝构造函数进行
        }
        void spilt() override{

        }
        TextSpilter(const TextSpilter& spilt){ // 拷贝构造函数
            // 进行处理
        }
        TextSpilter(){}
        
    };

    int main(){
        ISpilter* protoType = new TextSpilter(); //外界传入 protoType 

        ISpilter* spilt1 = protoType->clone();
        spilt1->spilt();

        delete protoType;
        delete spilt1;
        
        return 0;
    }
  ```
- **构建器(Builder)**
  建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
  一个 Builder 类会一步一步构造最终的对象。该 Builder 类是独立于其他对象的。
  ```cpp
  #include <iostream>
    #include <type_traits>

    class House{
        public:
        void show(){
            std::cout << "House" << std::endl;
        }
    };

    class HouseBuilder{  // 变化的部分对象构建
        public:
        House* GetBuildResult(){
            return pHouse;
        }
        virtual ~HouseBuilder(){}
        protected:
        House* pHouse;
        public:
        virtual void BuildPart1() = 0;
        virtual bool BuildPart2() = 0;
        virtual void BuildPart3() = 0;
    };

    class StoneHouse : public House{
        public:
        void show(){
            std::cout << " stoneHouse" << std::endl;
        }
    };

    class StoneHouseBuilder: public HouseBuilder {
        public:
        void BuildPart1() override{
            std::cout << "BuildPart1 " << std::endl;
        }
        bool BuildPart2() override{
            std::cout << "BuildPart2 " << std::endl;
            return true;
        }
        void BuildPart3() override{
            std::cout << "BuildPart3 " << std::endl;
        }
    };

    class  HouseDirector{
    public:
        HouseDirector(HouseBuilder* pHouseBuilder){
            this->pHouseBuilder = pHouseBuilder;
        }
        House* Construct(){
            pHouseBuilder->BuildPart1();
            bool flg = pHouseBuilder->BuildPart2();
            if(flg)
            {
                pHouseBuilder->BuildPart3();
            }
            return pHouseBuilder->GetBuildResult();
        }
    private:
        HouseBuilder* pHouseBuilder;
    };

    int main(){
        HouseBuilder* pHouseBuilder = new StoneHouseBuilder();
        HouseDirector* pDirector = new HouseDirector(pHouseBuilder);
        pDirector->Construct()->show();

        delete pHouseBuilder;
        delete pDirector;

        return 0;
    }
  ```
- **单例模式(Singleton)**
  单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
  这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象
  ```cpp
    #include <iostream>

    class Singletion{
        public:
        static Singletion* getInstanse();
        private:
        static Singletion* m_instanse;

        Singletion(){};
        Singletion(const Singletion&) = delete;
        Singletion& operator=(const Singletion&) = delete;

    };
    Singletion* Singletion::m_instanse = nullptr;

    //非线程安全实现
    Singletion* Singletion::getInstanse(){
        if (m_instanse == nullptr) {
            m_instanse = new Singletion();
        }

        return m_instanse;
    }

    int main(){
        Singletion* instanse = Singletion::getInstanse();
        Singletion* instanse1 = Singletion::getInstanse();
        std::cout << "instanse" << instanse << std::endl;
        std::cout << "instanse1" << instanse1 << std::endl;

        delete instanse1;

        return 0;
    }
  ```
  **加锁，但锁的代价过高**
  ```cpp
    #include <iostream>
    #include <atomic>
    #include <mutex>

    class Singletion{
        public:
        static Singletion* getInstanse();
        private:
        static Singletion* m_instanse;
        static std::mutex m_mutex;

        Singletion(){};
        Singletion(const Singletion&) = delete;
        Singletion& operator=(const Singletion&) = delete;

    };
    Singletion* Singletion::m_instanse = nullptr;
    std::mutex  Singletion::m_mutex;

    Singletion* Singletion::getInstanse(){
        std::lock_guard<std::mutex> lk{m_mutex};
        if (m_instanse == nullptr) {
            
            m_instanse = new Singletion();
        }

        return m_instanse;
    }

    int main(){
        Singletion* instanse = Singletion::getInstanse();
        Singletion* instanse1 = Singletion::getInstanse();
        std::cout << "instanse" << instanse << std::endl;
        std::cout << "instanse1" << instanse1 << std::endl;

        std::cout << " instanse == instanse1 \n" << (bool)(instanse1 == instanse) << std::endl;
        delete instanse1;

        return 0;
    }
  ```
  **双检查（锁），但是由于cpu指令重排导致问题**
  ```cpp
    #include <iostream>
    #include <atomic>
    #include <mutex>

    class Singletion{
        public:
        static Singletion* getInstanse();
        private:
        static Singletion* m_instanse;
        static std::mutex m_mutex;

        Singletion(){};
        Singletion(const Singletion&) = delete;
        Singletion& operator=(const Singletion&) = delete;

    };
    Singletion* Singletion::m_instanse = nullptr;
    std::mutex  Singletion::m_mutex;

    Singletion* Singletion::getInstanse(){
        if (m_instanse == nullptr) {
            std::lock_guard<std::mutex> lk{m_mutex};
            if (m_instanse == nullptr) {
                m_instanse = new Singletion();
            }
            
        }

        return m_instanse;
    }

    int main(){
        Singletion* instanse = Singletion::getInstanse();
        Singletion* instanse1 = Singletion::getInstanse();
        std::cout << "instanse" << instanse << std::endl;
        std::cout << "instanse1" << instanse1 << std::endl;

        std::cout << " instanse == instanse1 \n" << (bool)(instanse1 == instanse) << std::endl;
        delete instanse1;

        return 0;
    }
  ```
  **使用静态局部变量版本**
  ```cpp
    #include <iostream>
    #include <ostream>

    class Singletion{
        public:
        static Singletion& getInstanse(){
            static Singletion instanse;
            return instanse;
        };
        private:
        Singletion(){};
        Singletion(const Singletion&) = delete;
        Singletion& operator=(const Singletion&) = delete;

    };


    int main(){
        Singletion& instanse = Singletion::getInstanse();
        Singletion& instanse1 = Singletion::getInstanse();

        std::cout << "instanse ptr is :" << &instanse << std::endl;
        std::cout << "instanse1 ptr is ：" << &instanse1 << std::endl;

        return 0;
    }
  ```

  **C++11 之后的版本(原子语句)**
  ```cpp
    #include <iostream>
    #include <atomic>
    #include <mutex>

    class Singletion{
        public:
        static Singletion* getInstanse();
        private:
        static std::atomic<Singletion*> m_instanse;
        static std::mutex m_mutex;

        Singletion(){};
        Singletion(const Singletion&) = delete;
        Singletion& operator=(const Singletion&) = delete;

    };
    std::atomic<Singletion*> Singletion::m_instanse;
    std::mutex  Singletion::m_mutex;

    Singletion* Singletion::getInstanse(){
        Singletion* tmp = m_instanse.load(std::memory_order_relaxed);
        std::atomic_thread_fence(std::memory_order_acquire);//获取内存fence
        if (tmp == nullptr) {
            std::lock_guard<std::mutex> lock{m_mutex};
            tmp = m_instanse.load(std::memory_order_relaxed);
            if(tmp == nullptr)
            {
                tmp = new Singletion();
                std::atomic_thread_fence(std::memory_order_release); //释放内存
                m_instanse.store(tmp, std::memory_order_relaxed);
            }
            
        }

        return tmp;
    }

    int main(){
        Singletion* instanse = Singletion::getInstanse();
        Singletion* instanse1 = Singletion::getInstanse();
        std::cout << "instanse" << instanse << std::endl;
        std::cout << "instanse1" << instanse1 << std::endl;

        std::cout << " instanse == instanse1 \n" << (bool)(instanse1 == instanse) << std::endl;
        delete instanse1;

        return 0;
    }
  ```
- **享元模式(Flyweight)**
  面向对象很好地解决了"抽象”的问题，但是必不可免地要付出一定的代价。对于通常情况来讲,面向对象的成本大都可以忽略不计。但是某些情况,面向对象所带来的成本必须谨慎处理。享元模式：运用共享技术有效地支持大量细粒度的对象。
  ```cpp
    #include <iostream>
    #include <map>
    #include <string>

    class Font{
        public:
        Font(std::string k);
        private:
        std::string key;

    };

    class FontFactory{
        public:
        Font* GetFont(const std::string &k){
            std::map<std::string, Font*>::iterator itr = fontPool.find(k);
            if(itr != fontPool.end())
            {
                return fontPool[k];
            }
            else {
                Font *font = new Font(k);
                fontPool[k] = font;
                return font;
            }
        }
        void clear()
        {

        }

        private:
        std::map<std::string, Font *> fontPool;
    };
  ```
- **门面模式（Facade）**
  在组件构建过程中,某些接口之间直接的依赖常常会带来很多问题、甚至根本无法实现。采用添加一层间接(稳定)接口，来隔离本来互相紧密关联的接口是一种常见的解决方案。
  为了子系统中的一组接口提供一个一致（稳定）的界面，Facade模式定义了一个高层接口这一接口使得这一子系统更加容易使用（复用）。
  实际上Facade模式他体现的是一种设计原则和一种思想的表达，也就是在子系统内部和外部一种解耦的方式。载现实的开发过程中在不同的场景里面Facade模式表达方式差异很大，但要理解它的display在哪，所谓display就是说要把它子系统的变化给它圈住。
  ```cpp
    // 具体设计
  ```
- **代理模式(Proxy)**
  在组件构建过程中,某些接口之间直接的依赖常常会带来很多问题、甚至根本无法实现。采用添加一层间接(稳定)接口，来隔离本来互相紧密关联的接口是一种常见的解决方案。
  增加一层间接层”是软件系统中对许多复杂问题的一种常见解决方法。在面向对象系统中，直接使用某些对象会带来很多问题作为间接层的proxy对象便是解决这一问题的常用手段。
  具体proxy设计模式的实现方法、实现粒度都相差很大,有些可能对单个对象做细粒度的控制(如copy-on-write技术,有些可能对组件模块提供抽象代理层，在架构层次对对象做proxy。
  Proxy并不一定要求保持接口完整的一致性,只要能够实现间接控制,有时候损及一些透明性是可以接受的。
  ```cpp
        #include <iostream>

    class ISubject{
        public:
        virtual void Run() = 0;
        virtual ~ISubject(){}
    };

    class RealSubject: public ISubject{
        public:
        void Run() override{
            std::cout << "System run" << std::endl;
        }
    };

    class RealProxy : public ISubject{
        public:
        RealProxy(std::string userName, std::string passWord)
        :m_passWord(passWord),
        m_userName(userName),
        m_real{new RealSubject()}
        {

        }
        void Run() override{
            std::cout << "System Proxy run" << std::endl;
            if(checkUserNameAndPassword()){
                m_real->Run();
            }else {
                std::cout << "System check error" << std::endl;
            }
        }
        ~RealProxy(){
            if(m_real != nullptr)
            {
                delete m_real;
            }
        }
        private:
        bool checkUserNameAndPassword()
        {
            return  true;
        }

        RealSubject* m_real;
        std::string m_userName;
        std::string m_passWord;
    };

    int main(){
        RealProxy proxy("user", "pwd");
        proxy.Run();
        return 0;
    }
  ```
- **适配器(Adapter)**
  在组件构建过程中,某些接口之间直接的依赖常常会带来很多问题、甚至根本无法实现。采用添加一层间接(稳定)接口，来隔离本来互相紧密关联的接口是一种常见的解决方案。
  在软件系统中,由于应用环境的变化，常常需要将“一些现存的对象”放在新的环境中应用,但是新环境要求的接口是这些现存对象所不满足的。
  如何应对这种“迁移的变化”? 如何既能利用现有对象的良好实现,同时又能满足新的应用环境所要求的接口?
  将一个类的接口转换成客户希望的另，个接口。Adapter模式使得原本由于接台不兼容而不能一起土作的那些类可以一竖作。
  ```cpp
    #include <iostream>

    class ITarget{
        public:
        virtual void process() = 0;
        virtual ~ITarget(){}
    };

    // 遗留接口
    class IAdaptee{
        public:
        virtual int bar() = 0;
        virtual void foo(int) = 0;
        virtual ~IAdaptee(){}
    };

    // 遗留类型
    class OldClass: public IAdaptee{
        public:
        int bar() override{
            return 1024;
        }
        void foo(int data) override{
            std::cout << data << std::endl;
        }
    };

    // 对象适配器 
    class Adapter : public ITarget{
        public:
        Adapter(IAdaptee * pAdaptee)
        :pAdaptee_(pAdaptee)
        {

        }
        void process() override{
            int data = pAdaptee_->bar();
            pAdaptee_->foo(data);
        }
        protected:
        IAdaptee* pAdaptee_ ;  
    };

    int main(){
        IAdaptee * adaptee = new OldClass();
        ITarget* pTarget = new Adapter(adaptee);
        pTarget->process();
        return 0;
    }
  ```
- **中介者模式(Mediator)**
  在组件构建过程中,某些接口之间直接的依赖常常会带来很多问题、甚至根本无法实现。采用添加一层间接(稳定)接口，来隔离本来互相紧密关联的接口是一种常见的解决方案。
  在软件构建过程中,经常会出现多个对象互相关联交互的情况，对象之间常常会维持一种复杂的引用关系 ，如果遇到一-些需求的更改,这种直接的引|用关系将面临不断的变化。
  在这种情况下,我们可使用一个“中介对象”来管理对象间的关联关系,避免相互交互的对象之间的紧耦合引用关系,从而更好地抵御变化。
  **一个中介对象来封装(封装变化)一系列的对象交互。中介者使各对象不需要显式的相互引|用(编译时依赖>运行时依赖) ,从而使其耦合松散(管理变化)，而且可以独立地改变它们之间的交互。**
  ```cpp
  ```
- **状态模式（State）**
  在组件构建过程中，某些对象的状态经常面临变化，如何对这些变化进行有效的管理？同时又维持高层模块的稳定？“状态变化”模式为这一问题提供了一种解决方案。
  在软件构建过程中,某些对象的状态如果改变,其行为也会随之而发生变化,比如文档处于只读状态，其支持的行为和读写状态支持的行为就可能完全不同。
  如何在运行时根据对象的状态来透明地更改对象的行为?而不会为对象操作和状态转化之间引入紧耦合?
  允许一个对象在其内部状态改变时改变它的行为。从而使对象看起来似乎修改了其行为。
  State模式将所有与一个特定状态相关的行为都放入一个State的子类对象中,在对象状态切换时,切换相应的对象;但同时维持State的接口,这样实现了具体操作与状态转换之间的解耦。
  为不同的状态引入不同的对象使得状态转换变得更加明确,而且可以保证不会出现状态不一致的情况 ,因为转换是原子性的一即要么彻底转换过来,要么不转换。
  如果State对象没有实例变量,那么各个.上下文可以共享同一个State对象,从而节省对象开销。
  ```cpp
    class NetworkState{

    public:
        NetworkState* pNext;
        virtual void Operation1()=0;
        virtual void Operation2()=0;
        virtual void Operation3()=0;

        virtual ~NetworkState(){}
    };


    class OpenState :public NetworkState{
        
        static NetworkState* m_instance;
    public:
        static NetworkState* getInstance(){
            if (m_instance == nullptr) {
                m_instance = new OpenState();
            }
            return m_instance;
        }

        void Operation1(){
            
            //**********
            pNext = CloseState::getInstance();
        }
        
        void Operation2(){
            
            //..........
            pNext = ConnectState::getInstance();
        }
        
        void Operation3(){
            
            //$$$$$$$$$$
            pNext = OpenState::getInstance();
        }
        
        
    };

    class CloseState:public NetworkState{ }
    //...


    class NetworkProcessor{
        
        NetworkState* pState;
        
    public:
        
        NetworkProcessor(NetworkState* pState){
            
            this->pState = pState;
        }
        
        void Operation1(){
            //...
            pState->Operation1();
            pState = pState->pNext;
            //...
        }
        
        void Operation2(){
            //...
            pState->Operation2();
            pState = pState->pNext;
            //...
        }
        
        void Operation3(){
            //...
            pState->Operation3();
            pState = pState->pNext;
            //...
        }
    };

  ```
- **备忘录模式(Memento)** ***已过时***
  在组件构建过程中，某些对象的状态经常面临变化，如何对这些变化进行有效的管理？同时又维持高层模块的稳定？“状态变化”模式为这一问题提供了一种解决方案。
  在不破坏封装性的前提下,捕获一个对象的内部状态, 并在该对象之外保存这个状态。这样以后就可以将该对象恢复到原先保存的状态。
  备忘录( Memento )存储原发器( Originator )对象的内部状态，在需要时恢复原发器状态。
  Memento模式的核心是信息隐藏，即Originator需要向外接隐藏信息，保持其封装性。但同时又需要将状态保持到外界( Memento )。
  由于现代语言运行时(如C#、Java等 )都具有相当的对象序列化支持,因此往往采用效率较高、又较容易正确实现的序列化方案来实现Memento模式

  ```cpp
    #include<iostream>

    class Memento
    {
        private:
            std::string state_;
            //...
        public:
            Memento(std::string& state) :state_(state) {}
            std::string getState()const { return state_; }
            void setState(std::string& state) { state_ = state; }
    };

    class Originator
    {
        private:
            std::string state_;
            //...
        public:
            Originator() {}
            Memento createMomento()
            {
                Memento m(state_);
                return m;
            }
            void setMomento(const Memento& m)
            {
                state_ = m.getState();
            }
    };


    int main()
    {
        Originator orginator;
        //捕获对象状态，存储到备忘录
        Memento m = orginator.createMomento();
        //... 改变orginator状态

        //从备忘录中恢复
        orginator.setMomento(m);	

        return 0;
    }

  ```
- **组合模式(Composite)**
  常常有一-些组件在内部具有特定的数据结构，如果让客户程序依赖这些特定的数据结构，将极大地破坏组件的复用。这时候，将这些特定数据结构封装在内部,在外部提供统一的接口,来实现与特定数据结构无关的访问,是一种行之有效的解决方案。
  在软件在某些情况下,客户代码过多地依赖于对象容器复杂的内部实现结构,对象容器内部实现结构(而非抽象接口)的变化将引起客户代码的频繁变化,带来了代码的维护性、扩展性等弊端。
  如何将“客户代码与复杂的对象容器结构”解耦?让对象容器自己来实现自身的复杂结构，从而使得客户代码就像处理简单对象一样来处理复杂的对象容器?
  将对象组合成树形结构以表示“部分整体”的层次结构。Composite使得用户对单个对象和组合对象的使用具有一致性(稳定)。
  Composite模式采用树形结构来实现普遍存在的对象容器,从而，将”一对多”的关系转化为“一对一”的关系使得客户代码可以一致地(复用)处理对象和对象容器，无需关心处理的是单个的对象，还是组合的对象容器。
  将“客户代码与复杂的对象容器结构”解耦是Composite的核心思想，解耦之后，客户代码将与纯粹的抽象接口一”而非对象容器的内部实现结构二发生依赖 ,从而更能“应对变化”
  Composite模式在具体实现中,可以让父对象中的子对象反向追溯;如果父对象有频繁的遍历需求，可使用缓存技巧来改善效率。

  ```cpp
    #include<list>
    #include<string>
    #include<memory>
    #include<algorithm>
    #include<iostream>

    class Component
    {
    protected:
        std::string name;
    public:
        Component(std::string str) :name(str) {}
        virtual void display() = 0;
        virtual void add(Component*) = 0;
        virtual void remove(Component*) = 0;
        virtual ~Component() {}
    };

    class Composite:public Component
    {
    private:
        std::list<std::shared_ptr<Component>>elements;
    public:
        Composite(std::string str) :Component(str) {}
        void add(Component* element)
        {
            auto it = std::find_if(elements.begin(), elements.end(), 
                [element](std::shared_ptr<Component>ptr) {return element == ptr.get(); });
            if (it == elements.end())
            {
                elements.push_back(std::shared_ptr<Component>(element));
            }
        }
        void remove(Component* element)
        {
            auto it = std::find_if(elements.begin(), elements.end(),
                [element](std::shared_ptr<Component>ptr) {return element == ptr.get(); });
            if (it == elements.end())
            {
                return;
            }
            elements.erase(it);
        }
        void display()
        {
            for (auto it = elements.cbegin(); it != elements.cend(); ++it)
            {
                (*it)->display();   //多态调用
            }
        }
    };

    class Leaf : public Component 
    {
    public:
        Leaf(std::string str) :Component(str) {}
        void display()
        {
            std::cout << name << std::endl;
        }
        void add(Component* element)
        {
            std::cout << "Leaf cannot add" << std::endl;
        }
        void remove(Component* element)
        {
            std::cout << "Leaf cannot remove" << std::endl;
        }
    };


    int main()
    {
        Component* p = new Composite("北京");
        p->add(new Leaf("紫荆城"));
        p->add(new Leaf("朝阳区"));
        Component* p1 = new Composite("郑州");
        p1->add(new Leaf("郑大"));
        p1->add(new Leaf("。。。"));
        p->add(p1);
        p->display();
        return 0;
    }

  ```
- **迭代器(Iterator)**
  在软件构建过程中,集合对象内部结构常常变化各异。但对于这些集合对象,我们希望在不暴露其内部结构的同时,可以让外部客户代码透明地访问其中包含的元素;同时这种“透明遍历”也为“同一种算法在多种集合对象上进行操作”提供了可能。
  使用面向对象技术将这种遍历机制抽象为"迭代器对象”为“应对变化中的集合对象”提供了一种优雅的方式。
  迭代抽象:访问一-个聚合对象的内容而无需暴露它的内部表示。
  迭代多态:为遍历不同的集合结构提供一个统一-的接口 ,从而支持同样的算法在不同的集 合结构上进行操作。
  迭代器的健壮性考虑:遍历的同时更改迭代器所在的集合结构,会导致问题。
  ***对于C++来说这个设计模式已经过时了，C++已经有STL，它采用模板方式来实现，而模板是编译时多态，不是运行时多态，相比之下少了性能消耗。***
  ```cpp
    template<typename T>
    class Iterator
    {
    public:
        virtual void first() = 0;
        virtual void next() = 0;
        virtual bool isDone() const = 0;
        virtual T& current() = 0;
    };

    template<typename T>
    class MyCollection{
        
    public:
        
        Iterator<T> GetIterator(){
            //...
        }
        
    };

    template<typename T>
    class CollectionIterator : public Iterator<T>{
        MyCollection<T> mc;
    public:
        
        CollectionIterator(const MyCollection<T> & c): mc(c){ }
        
        void first() override {
            
        }
        void next() override {
            
        }
        bool isDone() const override{
            
        }
        T& current() override{
            
        }
    };

    void MyAlgorithm()
    {
        MyCollection<int> mc;
        
        Iterator<int> iter= mc.GetIterator();
        
        for (iter.first(); !iter.isDone(); iter.next()){
            cout << iter.current() << endl;
        }   
    }

  ```
- **职责链模式(Chain of Resposibility)**
  在软件构建过程中, -个请求可能被多个对象处理,但是每个请求在运行时只能有一个接受者,如果显式指定,将必不可少地带来请求发送者与接受者的紧耦合。
  如何使请求的发送者不需要指定具体的接受者?让请求的接受者自己在运行时决定来处理请求 ,从而使两者解耦。
  Chain of Responsibility模式的应用场合在于“一个请求可能有，多个接受者,但是最后真正的接受者只有一个”, 这时候请求发送者与接受者的耦合有可能出现“变化脆弱”的症状,职责链的目的就是将二者解耦，从而更好地应对变化。
  应用了Chain of Responsibility模式后,对象的职责分派将更具灵活性。我们可以在运行时动态添加/修改请求的处理职责。
  如果请求传递到职责链的末尾仍得不到处理,应该有一个合理的，缺省机制。这也是每一个接受对象的责任，而不是发出请求的对象的责任。
  ```cpp
    #include<iostream>
    #include<string>

    enum class RequestType
    {
        REQ_HANDLER1,
        REQ_HANDLER2,
        REQ_HANDLER3
    };

    class Reqest
    {
    public:
        Reqest(const std::string& desc, RequestType type) :description(desc), reqType(type) {}
        RequestType getReqType()const { return reqType; }
        const std::string& getDescription()const { return description; }
    private:
        std::string description;
        RequestType reqType;
    };

    class ChainHandler
    {
    public:
        ChainHandler() :nextChain(nullptr) {}
        void setNextChain(ChainHandler* next) { nextChain = next; }

        void handle(const Reqest& req)
        {
            if (this->canHandleRequest(req))
            {
                this->processRequest(req);
            }
            else
            {
                this->sendReqestToNextHandler(req);
            }
        }

    protected:
        virtual bool canHandleRequest(const Reqest&) = 0;
        virtual void processRequest(const Reqest&) = 0;

    private:
        void sendReqestToNextHandler(const Reqest& req)
        {
            if (nextChain != nullptr)
            {
                nextChain->handle(req);
            }
        }

    private:
        ChainHandler* nextChain;
    };

    class Handler1 :public ChainHandler
    {
    public:
        virtual bool canHandleRequest(const Reqest& req)
        {
            return req.getReqType() == RequestType::REQ_HANDLER1;
        }
        virtual void processRequest(const Reqest& req)
        {
            std::cout << "Handler1 is handle reqest: " << req.getDescription() << std::endl;
        }
    };
    class Handler2 : public ChainHandler {
    protected:
        bool canHandleRequest(const Reqest& req) override
        {
            return req.getReqType() == RequestType::REQ_HANDLER2;
        }
        void processRequest(const Reqest& req) override
        {
            std::cout << "Handler2 is handle reqest: " << req.getDescription() << std::endl;
        }
    };

    class Handler3 : public ChainHandler {
    protected:
        bool canHandleRequest(const Reqest& req) override
        {
            return req.getReqType() == RequestType::REQ_HANDLER3;
        }
        void processRequest(const Reqest& req) override
        {
            std::cout << "Handler3 is handle reqest: " << reqc.getDescription() << std::endl;
        }
    };

    int main()
    {
        Handler1 h1;
        Handler2 h2;
        Handler3 h3;
        h1.setNextChain(&h2);
        h2.setNextChain(&h3);

        Reqest req("process task ... ", RequestType::REQ_HANDLER3);
        h1.handle(req);
        return 0;
    }

  ```
- **命令模式(Command)**
  在软件构建过程中，“行为请求者” 与"行为实现者”通常呈现一种“紧耦合”。但在某些场合一比如需 要对行为进行“记录、撤销/重(undo/redo)、事务”等处理，这种无法抵御变化的紧耦合是不合适的。
  在这种情况下,如何将“行为请求者”与“行为实现者”解耦?将一组行为抽象为对象，可以实现二者之间的松耦合。
  将一个请求(行为)封装为一个对象，从而使你可用不同的请求对客户进行参数化;对请求排队或记录请求日志,以及支持可撤销的操作。
  Command模式的根本目的在于将“行为请求者”与“行为实现者”解耦,在面向对象语言中,常见的实现手段是“将行为抽象为对象”
  实现Command接口的具体命令对象ConcreteCommand有时候根据需要可能会保存一些额外的状态信息。通过使用Composite模式 ,可以将多个”命令”封装为一个“复合命令”MacroCommand.
  Command模式与C++中的函数对象有些类似。但两者定义行为接口的规范有所区别: Command以面向对象中的’ 接口-实现”来定义行为接口规范,更严格,但有性能损失; C++函数对象以函数签名来定义行为接口规范,更灵活，性能更高。
  ```cpp
    #include<string>
    #include<iostream>
    #include<vector>

    class Command
    {
    public:
        virtual void execute() = 0;
        virtual ~Command() {}
    };

    class ConcreteCommand1 :public Command
    {
    public:
        ConcreteCommand1(const std::string& str) :arg(str) {}
        void execute() override
        {
            std::cout << "#1 process..." << arg << std::endl;
        }
    private:
        std::string arg;
    };

    class ConcreteCommand2 :public Command
    {
    public:
        ConcreteCommand2(const std::string& str) :arg(str) {}
        void execute() override
        {
            std::cout << "#2 process..." << arg << std::endl;
        }
    private:
        std::string arg;
    };

    class MacroCommand :public Command
    {
    public:
        void addCommand(Command* command) { commands.push_back(command); }
        void execute() override
        {
            for (auto& c : commands)
            {
                c->execute();
            }
        }
    private:
        std::vector<Command*>commands;
    };


    int main()
    {
        ConcreteCommand1 command1("Arg ###");
        ConcreteCommand2 command2("Arg $$$");

        MacroCommand macro;
        macro.addCommand(&command1);
        macro.addCommand(&command2);

        macro.execute();
        return 0;
    }

  ```
- **访问器(Visitor)**
  在软件构建过程中，由于需求的改变,某些类层次结构中常常需要增加新的行为(方法) , 如果直接在基类中做这样的更改,将会给子类带来很繁重的变更负担,甚至破坏原有设计。
  如何在不更改类层次结构的前提下，在运行时根据需要 透明地为类层次结构.上的各个类动态添加新的操作,从而避免，上述问题?
  表示一个作用于某对象结构中的各元素的操作。使得可以在不改变(稳定)各元素的类的前提下定义(扩展)作用于这些元素的新操作(变化)
  Visito:模式通过所谓双重分发(double dispath )来实现在不更改，(不添加新的操作-编译时）Elemen类层次结构的前提下在运行时透明地为类层次结构上的各个类动态添和新的操作（支持变化）。
  所谓双重分发即Visitor模式中间包括了两个多态分发（注意其中的多态机制）：第一个为accept方法的多态 辨析；第二个为visitElementX方法的多态辨析
  Visito模式最大缺点在于扩展类层次结构（增添新的Element子类）会导致Visito类的改变。因此Visito模式适用于Element类层次结构稳定，而其中的操作却经常面临频繁改动。
  ```cpp
    #include<iostream>
    using namespace std;

    class Visitor;
    class Element {
    public:
        virtual void accept(Visitor& visitor) = 0;//第一次多态辨析(找accept)
        virtual ~Element() {}
    };
    class ElementA :public Element {
    public:
        virtual void accept(Visitor& visitor) override; //第二次多态辨析(找visitElementA)
    };
    class ElementB :public Element {
    public:
        void accept(Visitor& visitor) override;
    };

    class Visitor {
    public:
        virtual void visitElementA(ElementA& element) = 0;
        virtual void visitElementB(ElementB& element) = 0;
        virtual ~Visitor() {}
    };

    void ElementA::accept(Visitor& visitor) {
        visitor.visitElementA(*this);//第二次多态辨析(找visitElementA)
    }
    void ElementB::accept(Visitor& visitor) {
        visitor.visitElementB(*this);
    }
    //=================
    //对行为进行更改
    class Visiter1 :public Visitor {
    public:
        void visitElementA(ElementA& element) override {
            cout << "Visitor1 process ElementA" << endl;
        }
        void visitElementB(ElementB& element) override {
            cout << "Visitor1 process ElementB" << endl;
        }
    };

    class Visiter2 :public Visitor {
    public:
        void visitElementA(ElementA& element) override {
            cout << "Visitor2 process ElementA" << endl;
        }
        void visitElementB(ElementB& element) override {
            cout << "Visitor2 process ElementB" << endl;
        }
    };

    int main()
    {
        Visiter1 visitor;
        ElementA elementA;
        elementA.accept(visitor);//二次多态辨析

        ElementB elementB;
        elementB.accept(visitor);
        return 0;
    }


  ```
- **解析器(Interpreter)**
  在软件构建过程中,如果某一特定领域的问题比较复杂 ,类似的结构不断重复出现,如果使用普通的编程方式来实现将面临非常频繁的变化。
  在这种情况下,将特定领域的问题表达为某种语法规则下的句子，然后构建一个解释器来解释这样的句子,从而达到解决问题的目的。
  给定一个语言,定义它的文法的一种表示，并定义一种解释器，这个解释器使用该表示来解释语言中的句子。
  Interpreter模式的应用场合是Interpreter模式应用中的难点,只_有满足“业务规则频繁变化,且类似的结构不断重复出现，并且容易抽象为语法规则的问题”才适合使用Interpreter模式。
  使用Interpreter模式来表示文法规则,从而可以使用面向对象技巧来方便地"扩展”文法。
  Interpreter模式比较适合简单的文法表示,对于复杂的文法表示,Interperter模式会产生比较大的类层次结构, 需要求助于语法分析生成器这样的标准工具。
  ```cpp
    #include<iostream>
    #include<map>
    #include<string>
    #include<stack>

    class Expression
    {
    public:
        virtual int interpreter(std::map<char, int>) = 0;
        virtual ~Expression() {}
    };

    class VarExpression :public Expression
    {
    public:
        VarExpression(const char& k) :key(k) {}
        int interpreter(std::map<char, int> var) override
        {
            return var[key];
        }

    private:
        char key;
    };

    class SymbolExpression :public Expression
    {
    public:
        SymbolExpression(Expression* l, Expression* r) :left(l), right(r) {}
    protected:
        Expression* left;
        Expression* right;
    };

    class AddExpression : public SymbolExpression
    {
    public:
        AddExpression(Expression* left, Expression* right) :SymbolExpression(left, right) {}
        int interpreter(std::map<char, int> var) override
        {
            return left->interpreter(var) + right->interpreter(var);
        }
    };

    class SubExpression :public SymbolExpression
    {
    public:
        SubExpression(Expression* left, Expression* right) :SymbolExpression(left, right) {}
        int interpreter(std::map<char, int> var) override
        {
            return left->interpreter(var) - right->interpreter(var);
        }
    };
    class MulExpression :public SymbolExpression
    {
    public:
        MulExpression(Expression* left, Expression* right) :SymbolExpression(left, right) {}
        int interpreter(std::map<char, int> var) override
        {
            return left->interpreter(var) * right->interpreter(var);
        }
    };

    class DivExpression :public SymbolExpression
    {
    public:
        DivExpression(Expression* left, Expression* right) :SymbolExpression(left, right) {}
        int interpreter(std::map<char, int> var) override
        {
            return left->interpreter(var) / right->interpreter(var);
        }
    };

    Expression* analyse(std::string expStr)
    {
        std::stack<Expression*> expStack;
        Expression* left = nullptr;
        Expression* right = nullptr;
        for (int i = 0; i < expStr.size(); ++i)
        {
            switch (expStr[i])
            {
            case '+':
                // 加法运算
                left = expStack.top();
                right = new VarExpression(expStr[++i]);
                expStack.push(new AddExpression(left, right));
                break;
            case '-':
                // 减法运算
                left = expStack.top();
                right = new VarExpression(expStr[++i]);
                expStack.push(new SubExpression(left, right));
                break;
            case '*':
                // 乘法运算
                left = expStack.top();
                right = new VarExpression(expStr[++i]);
                expStack.push(new MulExpression(left, right));
                break;
            case '/':
                // 除法运算
                left = expStack.top();
                right = new VarExpression(expStr[++i]);
                expStack.push(new DivExpression(left, right));
                break;
            default:
                // 变量表达式
                expStack.push(new VarExpression(expStr[i]));
            }
        }
        Expression* expression = expStack.top();
        return expression;
    }

    void release(Expression* expression) {

        //释放表达式树的节点内存...
    }

    int main()
    {
        std::string expStr = "a+b-c+d-e*f/g";
        std::map<char, int> var;
        var.insert(std::make_pair('a', 5));
        var.insert(std::make_pair('b', 2));
        var.insert(std::make_pair('c', 1));
        var.insert(std::make_pair('d', 6));
        var.insert(std::make_pair('e', 10));
        var.insert(std::make_pair('f', 8));
        var.insert(std::make_pair('g', 4));

        Expression* expression = analyse(expStr);
        int result = expression->interpreter(var);
        std::cout << result << std::endl;

        release(expression);

        return 0;
    }

  ```