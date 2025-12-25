/* ================================================================================
 * 项目：银行账户管理系统（实践题1）
 * 
 * 涉及知识点：
 * ├─ 第7章：类 - 类的设计、构造函数、成员函数
 * ├─ 第8章：I/O库 - 文件读写、流操作（cin/cout/ifstream/ofstream）
 * ├─ 第9章：顺序容器 - vector容器存储多个账户和交易记录
 * ├─ 第10章：泛型算法 - find_if查找特定账户、sort排序账户
 * ├─ 第11章：关联容器 - （可选）使用map存储账户
 * ├─ 第12章：动态内存 - 指针、new/delete、智能指针（如需）
 * ├─ 第13章：拷贝控制 - 拷贝构造函数、赋值运算符、析构函数
 * └─ 第14章：运算符重载 - 重载+、-、==、<<等运算符
 * 
 * 题目要求：
 * ┌─ 1. 设计Account类
 * │  ├─ 成员变量：accountNumber(string)、name(string)、balance(double)、
 * │  │           transactionHistory(vector<string>)
 * │  ├─ 构造函数：支持初始化账户号、名字、初始金额
 * │  ├─ 拷贝构造函数：实现深拷贝
 * │  └─ 赋值运算符：处理自赋值和深拷贝
 * │
 * ├─ 2. 实现核心业务方法
 * │  ├─ deposit(double amount) - 存钱，记录交易时间和金额
 * │  ├─ withdraw(double amount) - 取钱，验证余额充足
 * │  ├─ transfer(Account &other, double amount) - 转账
 * │  ├─ getBalance() - 获取当前余额
 * │  └─ showTransactionHistory() - 显示所有交易记录
 * │
 * ├─ 3. 运算符重载
 * │  ├─ operator<< - 输出账户信息
 * │  ├─ operator+ - 存钱：account + 100
 * │  ├─ operator- - 取钱：account - 50
 * │  └─ operator== - 账户比较：按账户号判断
 * │
 * ├─ 4. 容器和算法应用
 * │  ├─ vector<Account> 存储多个账户
 * │  ├─ find_if 查找特定账户（按账户号）
 * │  └─ sort 按余额排序账户
 * │
 * └─ 5. I/O文件操作
 *    ├─ 从 accounts.txt 读取初始账户数据
 *    └─ 将交易记录输出到 transactions.log
 * 
 * 输入文件格式（accounts.txt）：
 * ACC001 张三 5000
 * ACC002 李四 8000
 * ACC003 王五 3000
 * 
 * 输出示例（transactions.log）：
 * 账户：ACC001 (张三)
 * 余额：$5000.00
 * 交易记录：
 * 2025-12-22 10:00 - 存入 $1000.00
 * 2025-12-22 10:15 - 取出 $500.00
 * 2025-12-22 10:30 - 转账给ACC002 $2000.00
 * ================================================================================ */

#include<iostream>
#include<string>
#include<vector>
#include<fstream>
#include<iomanip>
#include<algorithm>
#include<ctime>
#include<sstream>
#include<chrono>
#include<windows.h>

using namespace std;

/* ================================================================================
 * 【编程过程中遇到的主要问题及解决方案】
 * 
 * 1. 时间戳获取与格式化问题
 *    ├─ 问题：C++获取当前时间比较复杂，涉及多个步骤和类型转换
 *    ├─ 关键点：需要使用 <chrono> 库的 system_clock 获取时间点
 *    ├─ 难点1：time_point 需要转换为 time_t 才能格式化
 *    ├─ 难点2：Windows下必须使用 localtime_s（线程安全版本）而非 localtime
 *    ├─ 难点3：格式化需要 <iomanip> 的 std::put_time 配合 ostringstream
 *    └─ 解决：封装成 get_time() 方法，返回 "YYYY-MM-DD HH:MM:SS" 格式字符串
 * 
 * 2. 文件读写路径问题
 *    ├─ 问题：从不同目录运行程序时，相对路径会失效
 *    ├─ 场景：直接运行 .cpp 时路径是当前目录，但从 output 子目录运行时需要 ../
 *    ├─ 难点：ifstream/ofstream 打开失败不会抛异常，需要手动检查 if (!ifs)
 *    └─ 解决：尝试多个路径（当前目录和上级目录），并提供友好的错误提示
 * 
 * 3. 中文编码乱码问题
 *    ├─ 问题：Windows控制台默认使用GBK编码，而源文件是UTF-8编码
 *    ├─ 现象：中文输出显示为乱码（如"閾惰"而非"银行"）
 *    ├─ 尝试1：system("chcp 65001") - 会显示切换提示信息
 *    └─ 解决：使用 SetConsoleOutputCP(65001) - 需要 <windows.h> 头文件
 * 
 * 4. 构造函数与初始化列表
 *    ├─ 问题：Account 类没有默认构造函数，导致 Account acc1, acc2; 编译错误
 *    ├─ 原因：声明了带参数的构造函数后，编译器不再自动生成默认构造函数
 *    ├─ 错误信息：no matching function for call to 'Account::Account()'
 *    └─ 解决：使用拷贝构造初始化 Account acc1 = accounts[0]; 而非先声明后赋值
 * 
 * 5. const 成员函数问题
 *    ├─ 问题：在 const 对象或 const 引用参数中调用成员函数时编译错误
 *    ├─ 场景：saveTransactionsToFile(const vector<Account>&) 中调用 acc.get_time()
 *    ├─ 错误：passing 'const Account' as 'this' argument discards qualifiers
 *    └─ 解决：将不修改成员变量的方法声明为 const（如 get_time() const）
 * 
 * 6. 运算符重载与引用返回
 *    ├─ 问题：需要支持链式调用，如 acc.deposit(100).withdraw(50).getbalance()
 *    ├─ 关键：所有业务方法必须返回 Account& 而非 void
 *    ├─ 注意：operator+= 返回 *this，operator+ 返回临时对象（值传递）
 *    └─ 技巧：friend 函数可以访问私有成员，适合二元运算符重载
 * 
 * 7. 文件流的打开与关闭
 *    ├─ 问题：ifstream/ofstream 需要手动检查是否成功打开
 *    ├─ 读取：ifstream ifs(filename); if (!ifs) { 处理错误 }
 *    ├─ 写入：ofstream ofs(filename); 默认覆盖模式，ios::app 为追加模式
 *    ├─ 关闭：可以显式 ifs.close()，也可以依赖析构函数自动关闭
 *    └─ 技巧：while (ifs >> data) 自动处理文件末尾，失败时循环退出
 * 
 * 8. 字符串拼接与类型转换
 *    ├─ 问题：交易记录需要将时间、操作、金额拼接成字符串
 *    ├─ 方法1：to_string(amount) - 简单但格式不可控（如 1000.000000）
 *    ├─ 方法2：ostringstream + fixed + setprecision(2) - 精确控制小数位数
 *    └─ 建议：金额相关使用 ostringstream 确保格式统一（保留2位小数）
 * 
 * 9. vector 容器的使用
 *    ├─ 问题：需要在运行时动态添加交易记录
 *    ├─ 方法：vector<string> transaction_history; 支持 push_back() 动态扩容
 *    ├─ 遍历：for (const auto &record : transaction_history) - 范围for循环
 *    └─ 注意：拷贝构造和赋值运算符需要正确复制 vector（默认行为已足够）
 * 
 * 10. 友元函数与访问权限
 *     ├─ 问题：operator<< 需要访问私有成员，但不能作为成员函数（参数顺序）
 *     ├─ 原因：cout << acc 要求第一个参数是 ostream&，第二个是 Account&
 *     ├─ 解决：声明为 friend，在类外实现但能访问私有成员
 *     └─ 模式：friend ostream& operator<<(ostream& os, const Account& acc)
 * 
 * ================================================================================ */

// TODO: 在这里开始编写Account类和main函数
// 提示：
// 1. 先设计Account类的接口和成员变量
// 2. 实现构造函数、拷贝构造函数、赋值运算符、析构函数
// 3. 实现业务方法（存取转账）
// 4. 实现运算符重载
// 5. 在main函数中测试所有功能
class Account{
public:
    Account(const string &acc_num, const string &nm, double bal)
        : Account_number(acc_num), name(nm), balance(bal) {} // 原形式：string(acc_num)冗余转换，应直接传参
    Account(const Account &other)
        : Account_number(other.Account_number),
          name(other.name),
          balance(other.balance),
          transaction_history(other.transaction_history) {} // 原形式：在函数体中赋值，应用初始化列表效率更高
    Account& operator=(const Account &dress) { // 原形式：参数无const，缩进错误，if块内代码未正确缩进
        if (this != &dress) {
            Account_number = dress.Account_number;
            name = dress.name;
            balance = dress.balance;
            transaction_history = dress.transaction_history;
        }
        return *this;
    }
    
    // 【时间获取与格式化函数】
    // 目的: 获取当前系统时间并格式化为 "YYYY-MM-DD HH:MM:SS" 格式的字符串
    // 用途: 在交易记录中添加时间戳，记录每笔交易发生的准确时间
    inline std::string get_time() const {
        // 步骤1: 引入 chrono 命名空间，避免重复写 std::chrono::
        // chrono 是 C++11 引入的时间库，提供高精度时间操作
        using namespace std::chrono;
        
        // 步骤2: 获取当前时间点
        // system_clock::now() 返回当前系统时间的时间点(time_point)
        // auto 自动推导类型为 std::chrono::time_point<std::chrono::system_clock>
        auto tp = system_clock::now();
        
        // 步骤3: 将时间点转换为 time_t 类型
        // time_t 是 C 风格的时间类型，表示从1970年1月1日00:00:00至今的秒数
        // 需要这个转换是因为后续的格式化函数(put_time)需要 time_t 类型
        std::time_t tt = system_clock::to_time_t(tp);
        
        // 步骤4: 创建 tm 结构体用于存储本地时间
        // tm 结构体包含年、月、日、时、分、秒等时间字段
        // {} 表示值初始化，将所有字段初始化为0
        std::tm tm_buf{};
        
        // 步骤5: 将 time_t 转换为本地时间的 tm 结构体
        // localtime_s 是 Windows 下的线程安全版本(Linux 用 localtime_r)
        // 第一个参数: 输出参数，存储转换后的本地时间
        // 第二个参数: 输入参数，需要转换的 time_t 值
        // 功能: 将 UTC 时间转换为本地时区时间(如北京时间 UTC+8)
        localtime_s(&tm_buf, &tt);
        
        // 步骤6: 创建字符串输出流，用于拼接格式化的时间字符串
        // ostringstream 类似于 cout，但输出到字符串而非控制台
        std::ostringstream oss;
        
        // 步骤7: 将时间格式化并输出到字符串流
        // put_time(&tm_buf, 格式字符串) 按指定格式格式化时间
        // 格式说明: %Y-年份(4位) %m-月份(01-12) %d-日期(01-31)
        //          %H-小时(00-23) %M-分钟(00-59) %S-秒(00-59)
        // 示例输出: "2025-12-25 14:30:45"
        oss << std::put_time(&tm_buf, "%Y-%m-%d %H:%M:%S");
        
        // 步骤8: 返回字符串流的内容
        // str() 方法将 ostringstream 中的内容提取为 string 对象
        return oss.str();
    }

    

    friend std::ostream& operator<<(std::ostream& os, const Account& acc) { // 原形式：缺少空格（&operator）、字符串拼接过多、不应加endl
        // 【空格规范问题详解】原形式：friend std::ostream &operator <<
        // 问题1：& 紧跟在类型后，应为 std::ostream& 而非 std::ostream &
        // 问题2：operator 和 << 之间不应有空格，应为 operator<< 而非 operator <<
        // 一般规则：函数名前无空格，操作符名称不拆开。关键字后有空格（if, while, for等）
        os << acc.Account_number << " (" << acc.name << "): $" << acc.balance;
        return os;
    }
    Account& operator+=(double amount) { // 原形式：冗余this->，应直接用成员变量
        // 【this->冗余问题详解】原形式在多个地方出现：this->balance += amount
        // 问题：在成员函数内，不需要显式使用 this-> 来访问成员变量
        // 正确用法：balance += amount（直接使用）vs this->balance += amount（冗余）
        // 何时使用this->？仅在存在名字隐藏时（例如参数与成员变量同名）或模板中需要明确作用域时
        balance += amount;
        return *this;
    }
    friend Account operator+(Account ldress, double amount) { // 原形式：缺少空格、参数名ldress应用lhs
        ldress += amount;
        return ldress;
    }
    Account& operator-=(double amount) { // 原形式：缺少空格（if(）、冗余this->、代码缩进不规范
        // 【空格规范问题详解】原形式：if(this->balance<amount)
        // 问题1：if 后应有空格：if ( 而非 if(
        // 问题2：比较运算符两边应有空格：< 而非<，即 if (this->balance < amount)
        // 问题3：赋值运算符两边应有空格：= 而非=，即 balance = amount 而非 balance=amount
        // C++ 风格指南：所有二元运算符两边都应有空格，增强可读性
        if (balance < amount)
            cout << "余额不足，取款失败！" << endl;
        else
            balance -= amount;
        return *this;
    }
    friend Account operator-(Account ldress, double amount) { // 原形式：缺少空格、冗余this->、参数名不规范
        ldress -= amount;
        return ldress;
    }
    friend bool operator==(const Account& dress, const Account& other) { // 原形式：用check()自定义函数，应重载operator==
        return dress.Account_number == other.Account_number;
    }
    Account& deposit(double amount){
        balance += amount;
        transaction_history.push_back(get_time() + " - 存入 $" + to_string(amount));
        return *this;
    }
    Account& withdraw(double amount){
        if(balance < amount){
            std::cout<<"余额不足，取款失败！"<<std::endl;
        }
        else{
            balance -= amount;
            transaction_history.push_back(get_time() + " - 取出 $" + to_string(amount));
        }
        return *this;
    }
    Account& transfer(Account &other, double amount){
        if(balance < amount){
            std::cout<<"余额不足，转账失败!"<<std::endl;
        }
        else{
            balance -= amount;
            other.balance += amount;
            transaction_history.push_back(get_time() + " - 转账给" + other.Account_number + " $" + to_string(amount));
        }
        return *this;
    }
    Account& getbalance(){
        std::cout<<"当前余额：$"<<std::fixed<<std::setprecision(2)<<balance<<std::endl;
        return *this;
    }
    Account& showTransactionHistory(){
        std::cout<<"交易记录："<<std::endl;
        for(const auto &record : transaction_history){
            std::cout<<record<<std::endl;
        }
        return *this;
    }
    
    // 获取账户信息的getter方法
    string getAccountNumber() const { return Account_number; }
    string getName() const { return name; }
    double getBalanceValue() const { return balance; }
    const vector<string>& getTransactionHistory() const { return transaction_history; }
    
    // 将账户信息保存到文件
    void saveToFile(ofstream &ofs) const {
        ofs << "账户：" << Account_number << " (" << name << ")" << endl;
        ofs << "余额：$" << fixed << setprecision(2) << balance << endl;
        ofs << "交易记录：" << endl;
        for(const auto &record : transaction_history){
            ofs << record << endl;
        }
        ofs << "----------------------------------------" << endl;
    }
    
private:
    string Account_number;
    string name;
    double balance;
    vector<string> transaction_history;

    
};

// 从文件读取账户数据
vector<Account> loadAccountsFromFile(const string &filename) {
    vector<Account> accounts;
    ifstream ifs(filename);
    
    if (!ifs) {
        cout << "警告：无法打开文件 " << filename << "，使用默认账户数据" << endl;
        return accounts;
    }
    
    string accNum, name;
    double balance;
    
    while (ifs >> accNum >> name >> balance) {
        accounts.push_back(Account(accNum, name, balance));
    }
    
    ifs.close();
    cout << "成功从 " << filename << " 读取了 " << accounts.size() << " 个账户" << endl;
    return accounts;
}

// 将所有账户的交易记录保存到文件
void saveTransactionsToFile(const vector<Account> &accounts, const string &filename) {
    ofstream ofs(filename);
    
    if (!ofs) {
        cout << "错误：无法创建文件 " << filename << endl;
        return;
    }
    
    // 获取当前时间
    using namespace std::chrono;
    auto tp = system_clock::now();
    std::time_t tt = system_clock::to_time_t(tp);
    std::tm tm_buf{};
    localtime_s(&tm_buf, &tt);
    std::ostringstream oss;
    oss << std::put_time(&tm_buf, "%Y-%m-%d %H:%M:%S");
    
    ofs << "================ 银行账户交易记录 ================" << endl;
    ofs << "生成时间：" << oss.str() << endl << endl;
    
    for (const auto &acc : accounts) {
        acc.saveToFile(ofs);
    }
    
    ofs << "================ 记录结束 ================" << endl;
    ofs.close();
    cout << "交易记录已保存到 " << filename << endl;
}

int main()
{
    // 设置控制台输出为UTF-8编码，解决中文显示乱码问题
    SetConsoleOutputCP(65001);

    cout << "============ 银行账户管理系统测试 ============" << endl << endl;

    // 测试文件I/O功能(题目要求5)
    cout << "【测试0: 文件I/O功能】" << endl;

    // 尝试从文件读取账户(如果文件不存在会使用默认数据)
    // 先尝试当前目录, 再尝试上级目录(支持从output子目录运行)
    vector<Account> accounts = loadAccountsFromFile("accounts.txt");
    if (accounts.empty()) {
        // 如果当前目录未找到文件, 则尝试从上级目录读取
        accounts = loadAccountsFromFile("..\\accounts.txt");
    }

    // 如果文件读取成功且有足够账户, 使用文件数据; 否则使用默认数据
    if (accounts.size() < 3) {
        // 清空账户列表并添加默认账户数据
        accounts.clear();
        accounts.push_back(Account("ACC001", "张三", 5000));
        accounts.push_back(Account("ACC002", "李四", 8000));
        accounts.push_back(Account("ACC003", "王五", 3000));
        cout << "使用默认账户数据" << endl;
    } else {
        cout << "使用从文件读取的账户数据" << endl;
    }

    Account acc1 = accounts[0];
    Account acc2 = accounts[1];
    Account acc3 = accounts[2];
    cout << endl;

    // 1. 创建账户
    cout << "【测试1: 显示账户信息】" << endl;
    cout << "账户1: " << acc1 << endl;
    cout << "账户2: " << acc2 << endl;
    cout << "账户3: " << acc3 << endl << endl;

    // 2. 测试存款功能
    cout << "【测试2: 存款功能】" << endl;
    acc1.deposit(1000);
    cout << "张三存入 $1000 后:" << endl;
    acc1.getbalance();
    cout << endl;

    // 3. 测试取款功能
    cout << "【测试3: 取款功能】" << endl;
    acc1.withdraw(500);
    cout << "张三取出 $500 后:" << endl;
    acc1.getbalance();
    cout << endl;

    // 4. 测试取款失败(余额不足)
    cout << "【测试4: 余额不足场景】" << endl;
    acc3.withdraw(5000);  // 王五余额只有3000
    acc3.getbalance();
    cout << endl;

    // 5. 测试转账功能
    cout << "【测试5: 转账功能】" << endl;
    cout << "张三向李四转账 $2000" << endl;
    acc1.transfer(acc2, 2000);
    cout << "转账后余额:" << endl;
    acc1.getbalance();
    acc2.getbalance();
    cout << endl;

    // 6. 测试运算符重载 +
    cout << "【测试6: 运算符+ 存款】" << endl;
    acc3 = acc3 + 1500;
    cout << "王五使用 operator+ 存入 $1500 后:" << endl;
    acc3.getbalance();
    cout << endl;

    // 7. 测试运算符重载 -
    cout << "【测试7: 运算符- 取款】" << endl;
    acc3 = acc3 - 800;
    cout << "王五使用 operator- 取出 $800 后:" << endl;
    acc3.getbalance();
    cout << endl;

    // 8. 测试账户相等性比较
    cout << "【测试8: 账户比较】" << endl;
    Account acc1_copy("ACC001", "张三副本", 9999);
    if (acc1 == acc1_copy) {
        cout << "acc1 和 acc1_copy 账户号相同(相同账户)" << endl;
    }
    if (!(acc1 == acc2)) {
        cout << "acc1 和 acc2 账户号不同(不同账户)" << endl;
    }
    cout << endl;

    // 9. 显示交易历史
    cout << "【测试9: 查看交易历史】" << endl;
    cout << "\n张三的交易记录:" << endl;
    acc1.showTransactionHistory();
    cout << "\n李四的交易记录:" << endl;
    acc2.showTransactionHistory();
    cout << "\n王五的交易记录:" << endl;
    acc3.showTransactionHistory();
    cout << endl;

    // 10. 测试拷贝构造函数
    cout << "【测试10: 拷贝构造函数】" << endl;
    Account acc4 = acc1;
    cout << "拷贝后的账户:" << acc4 << endl;
    acc4.getbalance();
    cout << endl;

    // 11. 测试赋值运算符
    cout << "【测试11: 赋值运算符】" << endl;
    Account acc5("ACC005", "临时账户", 0);
    acc5 = acc2;
    cout << "赋值后的账户:" << acc5 << endl;
    acc5.getbalance();
    cout << endl;

    // 12. 链式调用测试
    cout << "【测试12: 链式调用】" << endl;
    Account acc6("ACC006", "赵六", 10000);
    cout << "连续操作: 存入$2000 -> 取出$500 -> 查看余额" << endl;
    acc6.deposit(2000).withdraw(500).getbalance();
    cout << endl;

    // 13. 保存交易记录到文件(题目要求5)
    cout << "【测试13: 保存交易记录到文件】" << endl;
    vector<Account> allAccounts = {acc1, acc2, acc3, acc4, acc5, acc6};
    // 如果从output子目录运行, 保存到上级目录
    string logFile = "transactions.log";
    saveTransactionsToFile(allAccounts, logFile);
    // 同时也保存到上级目录(如果在output子目录)
    saveTransactionsToFile(allAccounts, "..\\transactions.log");
    cout << endl;

    cout << "============ 所有测试完成 ============" << endl;
    cout << "提示: 查看 transactions.log 文件可以看到所有交易记录" << endl;

    return 0;
}
