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
user.h 
#ifndef USER_H
#define USER_H

#include <string>
#include <fstream>
#include <iostream>

using namespace std;

struct User {
    string username;
    string password;
};

bool registerUser();
bool loginUser();
bool checkUsernameExists(const string& username);

#endif // USER_H
————————————————————————————————————————————————————————

user.cpp 
#include "user.h"

bool checkUsernameExists(const string& username) {
    ifstream file("users.txt");
    if (!file.is_open()) return false;

    string line;
    while (getline(file, line)) {
        size_t pos = line.find(':');
        if (pos != string::npos && line.substr(0, pos) == username) {
            file.close();
            return true;
        }
    }
    file.close();
    return false;
}

bool registerUser() {
    string username, password;

    // 循环处理用户名输入，直到有效
    while (true) {
        cout << "请输入用户名: ";
        cin >> username;

        if (checkUsernameExists(username)) {
            char choice;
            cout << "错误：用户名已存在！" << endl;
            cout << "选择：1. 重新注册  2. 返回主菜单" << endl;

            while (true) {
                cin >> choice;
                if (choice == '1') break;        // 重新输入用户名
                else if (choice == '2') return false;  // 返回主菜单
                else cout << "无效选择！请输入 1 或 2：";
            }
        }
        else {
            break;  // 用户名有效，跳出循环
        }
    }

    // 读取密码
    cout << "请输入密码: ";
    cin >> password;

    // 保存到文件
    ofstream file("users.txt", ios::app);
    if (file.is_open()) {
        file << username << ":" << password << endl;
        file.close();
        cout << "注册成功！" << endl;
        return true;
    }
    else {
        cout << "错误：无法创建用户文件！" << endl;
        return false;
    }
}

bool loginUser() {
    string username, password;

    cout << "请输入用户名: ";
    cin >> username;
    cout << "请输入密码: ";
    cin >> password;

    ifstream file("users.txt");
    if (!file.is_open()) {
        cout << "错误：用户文件不存在！" << endl;
        return false;
    }

    string line;
    while (getline(file, line)) {
        size_t pos = line.find(':');
        if (pos != string::npos &&
            line.substr(0, pos) == username &&
            line.substr(pos + 1) == password) {
            file.close();
            cout << "登录成功！" << endl;
            return true;
        }
    }
    file.close();
    cout << "错误：用户名或密码错误！" << endl;
    return false;
}
————————————————————————————————————————————————————————
secondhand.cpp （main函数）
#include "user.h"

int main() {
    cout << "===== 校园二手交易平台 =====" << endl;

    while (true) {
        char choice;
        cout << "\n主菜单：" << endl;
        cout << "1. 注册" << endl;
        cout << "2. 登录" << endl;
        cout << "3. 退出" << endl;
        cout << "请选择：";
        cin >> choice;

        switch (choice) {
        case '1':
            registerUser();
            break;
        case '2':
            if (loginUser()) {
                cout << "欢迎使用校园二手交易平台！" << endl;
                // 这里可以添加登录后的功能代码
                return 0;  // 登录成功后退出程序，也可改为循环
            }
            break;
        case '3':
            cout << "感谢使用，再见！" << endl;
            return 0;
        default:
            cout << "无效选择，请重新输入！" << endl;
        }
    }

    return 0;
}
