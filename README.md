# Operating-System
Лабораторные работы по дисциплине "Операционные системы"
### Студент: Васильев Владислав Ильич
### Группа: 2261-ДБ

# Лабораторная работа №1
## Исследование компилятора gcc, ассемблера, процессов и синхронизации

## 1. Цель работы
Изучение компилятора GCC, ассемблерного кода, многопроцессного программирования и синхронизации в Linux.

## 2. Реализованная функция
Вычисление факториала числа (0-20) с использованием цикла.

## 3. Исследование оптимизаций GCC
Сам код написанный на C++
`#include <iostream>`
`#include <chrono>`
`
`unsigned long long factorial(int n) {`
`    unsigned long long result = 1;`
`    for (int i = 2; i <= n; ++i) {`
`        result *= i;`
`    }`
`    return result;`
`}`
`
`int main() {`
`    int number;`
`    std::cout << "Enter number (0-20): ";`
`    std::cin >> number;`
`
`    if (number < 0 || number > 20) {`
`        std::cerr << "Error: number must be between 0 and 20\n";`
`        return 1;`
`    }`
`
`    auto start = std::chrono::high_resolution_clock::now();`
`    unsigned long long result = factorial(number);`
`    auto end = std::chrono::high_resolution_clock::now();`
`
`    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);`
`
`    std::cout << "Factorial of " << number << " = " << result << "\n";`
`    std::cout << "Time: " << duration.count() << " microseconds\n";`
`
`    return 0;`
`}`
