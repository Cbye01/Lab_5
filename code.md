/////////////////////////////////////////////////////////////////////

#include <winsock2.h>
#include <iostream>

#pragma comment(lib, "ws2_32.lib")

void StartTCPServer(int port) {
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);

    SOCKET serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serverSocket == INVALID_SOCKET) {
        std::cerr << "Socket creation failed.\n";
        return;
    }

    sockaddr_in serverAddr = {};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(port);
    serverAddr.sin_addr.s_addr = INADDR_ANY;

    if (bind(serverSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
        std::cerr << "Bind failed.\n";
        closesocket(serverSocket);
        WSACleanup();
        return;
    }

    listen(serverSocket, SOMAXCONN);
    std::cout << "Server listening on port " << port << "\n";

    sockaddr_in clientAddr;
    int clientSize = sizeof(clientAddr);
    SOCKET clientSocket = accept(serverSocket, (sockaddr*)&clientAddr, &clientSize);
    if (clientSocket == INVALID_SOCKET) {
        std::cerr << "Accept failed.\n";
    } else {
        std::cout << "Client connected.\n";

        char buffer[1024] = {};
        int bytesReceived = recv(clientSocket, buffer, sizeof(buffer), 0);
        if (bytesReceived > 0) {
            std::cout << "Received: " << buffer << "\n";
            send(clientSocket, buffer, bytesReceived, 0);
        }
        closesocket(clientSocket);
    }

    closesocket(serverSocket);
    WSACleanup();
}

void StartTCPClient(const char* serverIP, int port) {
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);

    SOCKET clientSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientSocket == INVALID_SOCKET) {
        std::cerr << "Socket creation failed.\n";
        return;
    }

    sockaddr_in serverAddr = {};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(port);
    inet_pton(AF_INET, serverIP, &serverAddr.sin_addr);

    if (connect(clientSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
        std::cerr << "Connection failed.\n";
        closesocket(clientSocket);
        WSACleanup();
        return;
    }

    std::cout << "Connected to server.\n";
    const char* message = "Hello, Server!";
    send(clientSocket, message, strlen(message), 0);

    char buffer[1024] = {};
    int bytesReceived = recv(clientSocket, buffer, sizeof(buffer), 0);
    if (bytesReceived > 0) {
        std::cout << "Server responded: " << buffer << "\n";
    }

    closesocket(clientSocket);
    WSACleanup();
}

DWORD WINAPI ClientHandler(LPVOID clientSocket) {
    SOCKET client = *(SOCKET*)clientSocket;
    char buffer[1024] = {};
    int bytesReceived;

    while ((bytesReceived = recv(client, buffer, sizeof(buffer), 0)) > 0) {
        std::cout << "Client: " << buffer << "\n";
        send(client, buffer, bytesReceived, 0);
    }

    closesocket(client);
    return 0;
}

void StartMultiClientServer(int port) {
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);

    SOCKET serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    sockaddr_in serverAddr = {};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(port);
    serverAddr.sin_addr.s_addr = INADDR_ANY;

    bind(serverSocket, (sockaddr*)&serverAddr, sizeof(serverAddr));
    listen(serverSocket, SOMAXCONN);
    std::cout << "Multi-client server listening on port " << port << "\n";

    while (true) {
        sockaddr_in clientAddr;
        int clientSize = sizeof(clientAddr);
        SOCKET clientSocket = accept(serverSocket, (sockaddr*)&clientAddr, &clientSize);
        if (clientSocket != INVALID_SOCKET) {
            std::cout << "New client connected.\n";
            CreateThread(nullptr, 0, ClientHandler, &clientSocket, 0, nullptr);
        }
    }

    closesocket(serverSocket);
    WSACleanup();
}
void StartUDPServer(int port) {
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);

    SOCKET serverSocket = socket(AF_INET, SOCK_DGRAM, 0);
    sockaddr_in serverAddr = {};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(port);
    serverAddr.sin_addr.s_addr = INADDR_ANY;

    bind(serverSocket, (sockaddr*)&serverAddr, sizeof(serverAddr));
    std::cout << "UDP Server listening on port " << port << "\n";

    sockaddr_in clientAddr;
    int clientSize = sizeof(clientAddr);
    char buffer[1024];

    while (true) {
        int bytesReceived = recvfrom(serverSocket, buffer, sizeof(buffer), 0, (sockaddr*)&clientAddr, &clientSize);
        if (bytesReceived > 0) {
            buffer[bytesReceived] = '\0';
            std::cout << "Client: " << buffer << "\n";
            sendto(serverSocket, buffer, bytesReceived, 0, (sockaddr*)&clientAddr, clientSize);
        }
    }

    closesocket(serverSocket);
    WSACleanup();
}

void StartUDPClient(const char* serverIP, int port) {
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);

    SOCKET clientSocket = socket(AF_INET, SOCK_DGRAM, 0);
    sockaddr_in serverAddr = {};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(port);
    inet_pton(AF_INET, serverIP, &serverAddr.sin_addr);

    char buffer[1024];
    while (true) {
        std::cout << "Enter message: ";
        std::cin.getline(buffer, sizeof(buffer));
        sendto(clientSocket, buffer, strlen(buffer), 0, (sockaddr*)&serverAddr, sizeof(serverAddr));

        int serverSize = sizeof(serverAddr);
        int bytesReceived = recvfrom(clientSocket, buffer, sizeof(buffer), 0, (sockaddr*)&serverAddr, &serverSize);
        if (bytesReceived > 0) {
            buffer[bytesReceived] = '\0';
            std::cout << "Server: " << buffer << "\n";
        }
    }

    closesocket(clientSocket);
    WSACleanup();
}
