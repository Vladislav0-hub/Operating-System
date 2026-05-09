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
<pre>
#include &lt;iostream&gt;
#include &lt;chrono&gt;

unsigned long long factorial(int n) {
    unsigned long long result = 1;
    for (int i = 2; i &lt;= n; ++i) {
        result *= i;
    }
    return result;
}

int main() {
    int number;
    std::cout &lt;&lt; "Enter number (0-20): ";
    std::cin &gt;&gt; number;

    if (number &lt; 0 || number &gt; 20) {
        std::cerr &lt;&lt; "Error: number must be between 0 and 20\n";
        return 1;
    }

    auto start = std::chrono::high_resolution_clock::now();
    unsigned long long result = factorial(number);
    auto end = std::chrono::high_resolution_clock::now();

    auto duration = std::chrono::duration_cast&lt;std::chrono::microseconds&gt;(end - start);

    std::cout &lt;&lt; "Factorial of " &lt;&lt; number &lt;&lt; " = " &lt;&lt; result &lt;&lt; "\n";
    std::cout &lt;&lt; "Time: " &lt;&lt; duration.count() &lt;&lt; " microseconds\n";

    return 0;
}
&lt;&gt;
