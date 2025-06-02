# -
校园二手平台交易系统
钟佩琳 
阶段一交付物
- 可运行的注册登录控制台程序
- `users.txt`文件存储格式示例
- 主菜单交互流程演示
阶段二交付物
- 完整的商品管理功能（发布/浏览/搜索）
- 双文件数据存储（users.txt+goods.txt）
- 可演示的交易流程（从注册到搜索的完整闭环）


user.h 
#ifndef USER_H
#define USER_H

#include <string>
#include <fstream>
#include <iostream>
#include "goods.h" // 包含商品头文件
using namespace std;

class User {
private:
    string username;
    string password;

public:
    User() {}
    User(string un, string pwd) : username(un), password(pwd) {}

    bool registerUser() {
        if (checkUsernameExists(username)) {
            cout << "错误：用户名 \"" << username << "\" 已存在！\n";
            return false;
        }
        ofstream file("users.txt", ios::app);
        if (file << username << ":" << password << endl) {
            file.close();
            return true;
        }
        cout << "错误：无法保存用户数据！\n";
        return false;
    }

    bool loginUser() const {
        ifstream file("users.txt");
        string line, un, pwd;
        while (getline(file, line)) {
            if (line.find(username + ":") == 0) {
                pwd = line.substr(username.length() + 1);
                return pwd == password;
            }
        }
        cout << "错误：用户名或密码错误！\n";
        return false;
    }

    static bool checkUsernameExists(const string& un) {
        ifstream file("users.txt");
        string line;
        while (getline(file, line)) {
            if (line.find(un + ":") == 0) return true;
        }
        return false;
    }

    // 获取当前用户发布的商品
    vector
        <Goods> getPublishedGoods() const {
        return GoodsFile::getByPublisher(username);
    }
};

#endif // USER_H
————————————————————————————————————————————————————————
menu.h 
#ifndef MENU_H
#define MENU_H

#include "user.h"
#include "goods.h"
#include <iomanip>
using namespace std;

class Menu {
private:
    bool isLoggedIn;
    string currentUser;

public:
    Menu() : isLoggedIn(false), currentUser("") {}

    void run() {
        cout << "===== 二手交易平台 v1.0 =====" << endl;
        while (true) showMainMenu(), handleChoice();
    }

private:
    void showMainMenu() const {
        cout << "\n" << string(25, '=') << endl;
        if (!isLoggedIn) {
            cout << "1. 注册新用户\n2. 用户登录\n3. 浏览商品\n4. 退出系统" << endl;
        }
        else {
            cout << "1. 发布商品\n2. 我的商品\n3. 浏览商品\n4. 搜索商品\n5. 按类别浏览商品\n6. 退出登录\n7. 退出系统" << endl;
            cout << "当前用户：" << currentUser << endl;
        }
        cout << "请选择操作：";
    }

    void handleChoice() {
        int choice;
        cin >> choice;

        if (!isLoggedIn) {
            switch (choice) {
            case 1: registerUser(); break;
            case 2: loginUser(); break;
            case 3: browseGoods(); break;
            case 4: exitSystem(); return;
            default: invalidInput();
            }
        }
        else {
            switch (choice) {
            case 1: publishGoods(); break;
            case 2: myGoods(); break;
            case 3: browseGoods(); break;
            case 4: searchGoods(); break;
            case 5: browseGoodsByCategory(); break; // 新增：按类别浏览商品
            case 6: logout(); break;
            case 7: exitSystem(); return;
            default: invalidInput();
            }
        }
    }

    void browseGoods() const {
        vector<Goods> allGoods = GoodsFile::getAll();
        displayGoodsList(allGoods);
    }

    void publishGoods() const {
        Goods newGoods;
        cout << "===== 发布商品 =====" << endl;
        cout << "商品名称：";
        cin >> newGoods.name;
        cin.ignore(); // 清除缓冲区
        cout << "商品描述：";
        getline(cin, newGoods.description);

        string priceStr;
        do {
            cout << "商品价格：";
            cin >> priceStr;
        } while (!GoodsValidator::validatePrice(priceStr));
        newGoods.price = stod(priceStr);

        cout << "联系方式：";
        cin >> newGoods.contact;

        cout << "请选择商品类别：" << endl;
        cout << "1. 电子产品" << endl;
        cout << "2. 书籍" << endl;
        cout << "3. 服装" << endl;
        cout << "4. 体育用品" << endl;
        cout << "5. 生活用品" << endl;
        cout << "6. 其他" << endl;
        int categoryChoice;
        cin >> categoryChoice;
        switch (categoryChoice) {
        case 1: newGoods.category = ProductCategory::ELECTRONICS; break;
        case 2: newGoods.category = ProductCategory::BOOKS; break;
        case 3: newGoods.category = ProductCategory::CLOTHING; break;
        case 4: newGoods.category = ProductCategory::SPORTS; break;
        case 5: newGoods.category = ProductCategory::LIVING; break;
        case 6: newGoods.category = ProductCategory::OTHER; break;
        default:
            cout << "无效的选择，默认选择其他类别" << endl;
            newGoods.category = ProductCategory::OTHER;
        }

        newGoods.publisher = currentUser;  // 发布时自动填充当前用户名

        if (GoodsValidator::validate(newGoods)) {
            if (GoodsFile::save(newGoods)) {
                cout << "商品发布成功！" << endl;
            }
        }
    }

    void myGoods() const {
        User currentUserObj(currentUser, ""); // 创建临时用户对象
        vector<Goods> myGoods = currentUserObj.getPublishedGoods();

        if (myGoods.empty()) {
            cout << "您还没有发布任何商品" << endl;
            return;
        }

        cout << "===== 我的商品 =====" << endl;
        displayGoodsList(myGoods);
    }

    void searchGoods() const {
        string keyword;
        cout << "请输入搜索关键词：";
        cin >> keyword;
        transform(keyword.begin(), keyword.end(), keyword.begin(), ::tolower);

        vector<Goods> allGoods = GoodsFile::getAll();
        vector<Goods> result;

        for (const auto& goods : allGoods) {
            string lowerName = goods.name;
            transform(lowerName.begin(), lowerName.end(), lowerName.begin(), ::tolower);
            if (lowerName.find(keyword) != string::npos) {
                result.push_back(goods);
            }
        }

        if (result.empty()) {
            cout << "未找到匹配的商品" << endl;
        }
        else {
            cout << "===== 搜索结果 =====" << endl;
            displayGoodsList(result);
        }
    }

    void browseGoodsByCategory() const {
        cout << "请选择要浏览的商品类别：" << endl;
        cout << "1. 电子产品" << endl;
        cout << "2. 书籍" << endl;
        cout << "3. 服装" << endl;
        cout << "4. 体育用品" << endl;
        cout << "5. 生活用品" << endl;
        cout << "6. 其他" << endl;
        int categoryChoice;
        cin >> categoryChoice;
        ProductCategory category;
        switch (categoryChoice) {
        case 1: category = ProductCategory::ELECTRONICS; break;
        case 2: category = ProductCategory::BOOKS; break;
        case 3: category = ProductCategory::CLOTHING; break;
        case 4: category = ProductCategory::SPORTS; break;
        case 5: category = ProductCategory::LIVING; break;
        case 6: category = ProductCategory::OTHER; break;
        default:
            cout << "无效的选择，默认选择其他类别" << endl;
            category = ProductCategory::OTHER;
        }

        vector<Goods> categoryGoods = GoodsFile::getByCategory(category);

        if (categoryGoods.empty()) {
            cout << "该类别下没有商品" << endl;
        }
        else {
            cout << "===== " << categoryToString(category) << " 类别商品 =====" << endl;
            displayGoodsList(categoryGoods);
        }
    }

    void displayGoodsList(const vector<Goods>& goodsList) const {
        if (goodsList.empty()) {
            cout << "没有商品信息" << endl;
            return;
        }

        cout << setw(20) << "商品名称" << setw(30) << "商品描述" << setw(10)
            << "价格" << setw(20) << "联系方式" << setw(20) << "发布者" << setw(20) << "类别" << endl;
        cout << string(120, '-') << endl;

        for (const auto& goods : goodsList) {
            cout << setw(20) << goods.name << setw(30) << goods.description << setw(10)
                << goods.price << setw(20) << goods.contact << setw(20) << goods.publisher << setw(20) << categoryToString(goods.category) << endl;
        }
    }

    void registerUser() {
        string un, pwd;
        cout << "用户名："; cin >> un;
        cout << "密码："; cin >> pwd;
        User user(un, pwd);
        if (user.registerUser()) cout << "注册成功，请登录！\n";
    }

    void loginUser() {
        string un, pwd;
        cout << "用户名："; cin >> un;
        cout << "密码："; cin >> pwd;
        User user(un, pwd);
        if (user.loginUser()) {
            isLoggedIn = true;
            currentUser = un;
            cout << "登录成功！\n";
        }
    }

    void logout() {
        isLoggedIn = false;
        currentUser = "";
        cout << "您已退出登录！\n";
    }

    void exitSystem() {
        cout << "\n感谢使用，再见！\n";
        exit(0);
    }

    void invalidInput() {
        cout << "输入无效，请输入有效的数字！\n";
        cin.clear();
        cin.ignore(1000, '\n');
    }
};

#endif // MENU_H
—————————————————————————————————————————————————————————
goods.h 

#ifndef GOODS_H
#define GOODS_H

#include <string>
#include <fstream>
#include <iostream>
#include <stdexcept>
#include <vector>
#include <algorithm>
#include <unordered_map>

using namespace std;

// 商品类别枚举
enum class ProductCategory {
    ELECTRONICS, BOOKS, CLOTHING, SPORTS, LIVING, OTHER
};

// 枚举转换为字符串
string categoryToString(ProductCategory category);

// 字符串转换为枚举
ProductCategory stringToCategory(const string& str);

// 商品结构体定义
struct Goods {
    string name;        // 商品名称
    string description; // 商品描述
    double price;       // 商品价格
    string contact;     // 联系方式
    string publisher;   // 发布用户
    ProductCategory category; // 商品类别

    // 构造函数
    Goods() : price(0), category(ProductCategory::OTHER) {}
    Goods(string n, string d, double p, string c, string pub, ProductCategory cat)
        : name(n), description(d), price(p), contact(c), publisher(pub), category(cat) {
    }

    // 转换为存储字符串格式，例如:名称:描述:价格:联系方式:发布者:类别
    string toStorageString() const {
        return name + ":" + description + ":" + to_string(price) + ":" + contact + ":" + publisher + ":" + categoryToString(category);
    }

    // 从存储字符串创建商品对象，格式错误时抛出异常
    static Goods fromStorageString(const string& str) {
        Goods goods;
        size_t pos1 = str.find(':');
        size_t pos2 = str.find(':', pos1 + 1);
        size_t pos3 = str.find(':', pos2 + 1);
        size_t pos4 = str.find(':', pos3 + 1);
        size_t pos5 = str.find(':', pos4 + 1);  // 找到第五个冒号

        // 确保有5个冒号，即6个字段
        if (pos1 == string::npos || pos2 == string::npos || pos3 == string::npos || pos4 == string::npos || pos5 == string::npos) {
            throw invalid_argument("Invalid goods format (required: name:desc:price:contact:publisher:category)");
        }

        goods.name = str.substr(0, pos1);
        goods.description = str.substr(pos1 + 1, pos2 - pos1 - 1);

        // 价格转换
        try {
            goods.price = stod(str.substr(pos2 + 1, pos3 - pos2 - 1));
        }
        catch (const exception&) {
            throw invalid_argument("Invalid price format");
        }

        goods.contact = str.substr(pos3 + 1, pos4 - pos3 - 1);  // 联系方式
        goods.publisher = str.substr(pos4 + 1, pos5 - pos4 - 1);  // 发布者
        goods.category = stringToCategory(str.substr(pos5 + 1));  // 类别

        return goods;
    }
};  // 结构体定义结束

// 商品验证类
class GoodsValidator {
public:
    // 验证商品信息是否全部有效
    static bool validate(const Goods& goods) {
        if (goods.name.empty()) {
            cout << "错误：商品名称不能为空" << endl;
            return false;
        }
        if (goods.price < 0) {
            cout << "错误：商品价格不能为负数" << endl;
            return false;
        }
        if (goods.contact.empty()) {
            cout << "错误：联系方式不能为空" << endl;
            return false;
        }
        if (goods.publisher.empty()) {
            cout << "错误：发布者信息不能为空" << endl;
            return false;
        }
        return true;
    }

    // 验证价格字符串是否为有效数字
    static bool validatePrice(const string& priceStr) {
        if (priceStr.empty()) return false;

        // 检查是否为非负数字
        if (priceStr[0] == '-') {
            cout << "错误：价格不能为负数" << endl;
            return false;
        }
        if (priceStr.find_first_not_of("0123456789.") != string::npos) {
            cout << "错误：价格必须为数字" << endl;
            return false;
        }
        return true;
    }
};

// 商品文件管理类（供用户系统调用）
class GoodsFile {
private:
    static unordered_map<ProductCategory, vector<Goods>> categoryGoodsMap;
    static bool isInitialized;

    static void initCategoryGoodsMap();

public:
    // 保存商品信息到文件
    static bool save(const Goods& goods) {
        ofstream file("goods.txt", ios::app);
        if (!file.is_open()) {
            cout << "错误：无法打开商品文件" << endl;
            return false;
        }
        file << goods.toStorageString() << endl;
        file.close();
        categoryGoodsMap[goods.category].push_back(goods);
        return true;
    }

    // 获取所有商品信息
    static vector<Goods> getAll() {
        vector<Goods> goodsList;
        ifstream file("goods.txt");
        if (!file.is_open()) return goodsList;

        string line;
        while (getline(file, line)) {
            try {
                goodsList.push_back(Goods::fromStorageString(line));
            }
            catch (const exception&) {
                cout << "错误：文件格式不正确，跳过此商品记录" << endl;
            }
        }
        file.close();
        return goodsList;
    }

    // 根据发布者筛选商品信息
    static vector<Goods> getByPublisher(const string& username) {
        vector<Goods> publisherGoods;
        vector<Goods> allGoods = getAll();
        for (const auto& goods : allGoods) {
            if (goods.publisher == username) {  // 精确匹配发布者字段
                publisherGoods.push_back(goods);
            }
        }
        return publisherGoods;
    }

    // 根据类别筛选商品信息
    static vector<Goods> getByCategory(ProductCategory category) {
        initCategoryGoodsMap();
        return categoryGoodsMap[category];
    }
};

#endif // GOODS_H
—————————————————————————————————————————————————————————

goods.cpp 

#include "goods.h"
#include <iomanip>

// 商品列表展示函数
void displayGoodsList(ifstream& file) {
    if (!file.is_open()) {
        cout << "商品文件不存在，没有商品信息" << endl;
        return;
    }

    string line;
    int count = 0;

    cout << "\n========== 商品列表 ==========" << endl;
    cout << setw(20) << "商品名称" << setw(30) << "商品描述"
        << setw(10) << "价格" << setw(20) << "联系方式" << setw(20) << "类别" << endl;
    cout << string(120, '-') << endl;

    while (getline(file, line)) {
        try {
            Goods goods = Goods::fromStorageString(line);
            cout << setw(20) << goods.name << setw(30) << goods.description
                << setw(10) << goods.price << setw(20) << goods.contact << setw(20) << categoryToString(goods.category) << endl;
            count++;
        }
        catch (const exception& e) {
            cout << "警告：解析商品信息失败 - " << e.what() << endl;
        }
    }

    if (count == 0) {
        cout << "没有商品信息" << endl;
    }
    else {
        cout << "========== 共 " << count << " 件商品 ==========" << endl;
    }
}

// 保存商品到文件
bool saveGoodsToFile(const Goods& goods) {
    ofstream file("goods.txt", ios::app);
    if (!file.is_open()) {
        cout << "无法打开商品文件" << endl;
        return false;
    }

    file << goods.toStorageString() << endl;
    file.close();
    return true;
}

// 将枚举转换为字符串（正确位置：函数外部）
string categoryToString(ProductCategory category) {
    switch (category) {
    case ProductCategory::ELECTRONICS: return "电子产品";
    case ProductCategory::BOOKS: return "书籍";
    case ProductCategory::CLOTHING: return "服装";
    case ProductCategory::SPORTS: return "体育用品";
    case ProductCategory::LIVING: return "生活用品";
    case ProductCategory::OTHER: return "其他";
    default: return "未知类别";
    }
}

// 将字符串转换为枚举（正确位置：函数外部）
ProductCategory stringToCategory(const string& str) {
    if (str == "电子产品") return ProductCategory::ELECTRONICS;
    if (str == "书籍") return ProductCategory::BOOKS;
    if (str == "服装") return ProductCategory::CLOTHING;
    if (str == "体育用品") return ProductCategory::SPORTS;
    if (str == "生活用品") return ProductCategory::LIVING;
    if (str == "其他") return ProductCategory::OTHER;
    return ProductCategory::OTHER;
}

// 从文件读取所有商品
vector<Goods> readAllGoods() {
    vector<Goods> goodsList;
    ifstream file("goods.txt");
    if (!file.is_open()) {
        return goodsList;
    }

    string line;
    while (getline(file, line)) {
        try {
            goodsList.push_back(Goods::fromStorageString(line));
        }
        catch (const exception&) {
            continue; // 格式错误，跳过
        }
    }
    file.close();
    return goodsList;
}

// 定义静态成员变量
unordered_map
<ProductCategory, vector<Goods>> GoodsFile::categoryGoodsMap;
bool GoodsFile::isInitialized = false;

// 初始化分类映射（实现）
void GoodsFile::initCategoryGoodsMap() {
    if (isInitialized) return;
    vector
        <Goods> allGoods = getAll();
    for (const auto& goods : allGoods) {
        categoryGoodsMap
            [goods.category].push_back(goods);
    }
    isInitialized= true;
}

—————————————————————————————————————————————————————————

secondhand.cpp （main函数）
#include "menu.h"

int main() {
    Menu menu;
    menu.run();
    return 0;
}