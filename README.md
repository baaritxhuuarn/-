# -
校园二手平台交易系统
钟佩琳 第一阶段任务：（用户系统开发）
- [ ] 设计用户数据结构：在头文件中定义用户信息结构体（用户名、密码）
- [ ] 实现用户注册功能：
  - 控制台交互获取用户名/密码
  - 使用`fstream`将用户信息写入`users.txt`文件（格式：用户名:密码）
  - 增加重复用户名检测（读取文件比对）
- [ ] 编写登录验证函数：
  - 从文件读取用户数据进行匹配
  - 返回登录状态（布尔值）
  - 简单错误提示（用户名不存在/密码错误）
阶段交付物
- 可运行的注册登录控制台程序
- `users.txt`文件存储格式示例
- 主菜单交互流程演示

user.h 
#ifndef USER_H
#define USER_H

#include <string>
#include <fstream>
#include <iostream>
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
};

#endif // USER_H
————————————————————————————————————————————————————————
menu.h 
#ifndef MENU_H
#define MENU_H

#include "user.h"
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
            cout << "1. 发布商品\n2. 我的商品\n3. 浏览商品\n4. 退出登录\n5. 退出系统" << endl;
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
            case 3: browseGoods(); break; // 新增浏览功能（占位）
            case 4: exitSystem(); return;
            default: invalidInput();
            }
        }
        else {
            switch (choice) {
            case 1: publishGoods(); break; // 发布功能（占位）
            case 2: myGoods(); break;      // 我的商品（占位）
            case 3: browseGoods(); break;  // 浏览功能（保留）
            case 4: logout(); break;
            case 5: exitSystem(); return;
            default: invalidInput();
            }
        }
    }

    // 新增占位函数（阶段二实现）
    void browseGoods() const {
        cout << "【浏览商品】功能开发中，可查看所有发布的商品\n";
        // 阶段二从goods.txt读取并显示
    }

    void publishGoods() const {
        cout << "【发布商品】请输入商品信息（阶段二实现）\n";
    }

    void myGoods() const {
        cout << "【我的商品】查看您发布的所有商品（阶段二实现）\n";
    }

    // menu.h中预留接口
    void searchGoods() const {
        cout << "【搜索商品】请输入关键词（阶段二实现）\n";
        // 阶段二添加：
        // string keyword; cin >> keyword;
        // 从goods.txt中搜索含keyword的商品
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
        cout << "已退出登录！\n";
    }

    void exitSystem() {
        cout << "\n感谢使用，再见！\n";
        exit(0);
    }

    void invalidInput() {
        cout << "错误：请输入有效数字！\n";
        cin.clear();
        cin.ignore(1000, '\n');
    }
};

#endif // MENU_H

—————————————————————————————————————————————————————————
secondhand.cpp （main函数）
#include "menu.h"

int main() {
    Menu menu;
    menu.run();
    return 0;
}