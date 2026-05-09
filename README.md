# Operating-System
Лабораторные работы по дисциплине "Операционные системы"
### Студент: Васильев Владислав Ильич
### Группа: 2261-ДБ

# Лабораторная работа №1
## Исследование компилятора gcc, ассемблера, процессов и синхронизации

##Цель лабораторной 
Изучение компилятора GCC, ассемблерного кода, многопроцессного программирования и синхронизации в Linux.

##Реализованная функция
Вычисление факториала числа (0-20) с использованием цикла.

##Исследование оптимизаций GCC
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
</pre>

Все выполненные оптимизации
<pre>
g++ -S -o factorial_no_opt.s factorial.cpp - Без оптимизации

g++ -S -O1 -o factorial_O1.s factorial.cpp - 1-й уровень оптимизации
g++ -S -O2 -o factorial_O2.s factorial.cpp - 2-ой уровень оптимизации
g++ -S -O3 -o factorial_O3.s factorial.cpp - 3-й уровень оптимизации

g++ -S -g -o factorial_g.s factorial.cpp - C отладочной информацией
</pre>

На разбор я взял 2-ой уровень оптимизации(я закомментировал, в директории есть просто factorial_O2.s и закомментированный factorial_O2_commented.s)
<pre>
# ===================================================================
# ФАЙЛ: factorial_O2_commented.s
# ИСХОДНИК: factorial.cpp
# КОМПИЛЯТОР: g++ -S -O2 factorial.cpp (MinGW-w64 GCC 8.1.0)
# ФУНКЦИЯ: Вычисление факториала числа
# Выполнил: Васильев Владислав Ильич(2261-ДБ)
# ===================================================================

	.file	"factorial.cpp"              # Исходный файл
	.text                                 # Секция кода
	.p2align 4,,15                       # Выравнивание на 16 байт (макс 15 байт nop)

# ===================================================================
# СТАНДАРТНАЯ ФУНКЦИЯ ОЧИСТКИ (для глобалных объектов C++)
# Вызывается при завершении программы через atexit()
# ===================================================================
	.def	__tcf_0;	.scl	3;	.type	32;	.endef
	.seh_proc	__tcf_0
__tcf_0:
.LFB2410:                               # Local Function Begin
	.seh_endprologue
	leaq	_ZStL8__ioinit(%rip), %rcx   # RCX = адрес глобального ioinit
	jmp	_ZNSt8ios_base4InitD1Ev       # Вызов деструктора ios_base::Init
	.seh_endproc

# ===================================================================
# ФУНКЦИЯ factorial(int n)
# Вход: ECX = n (первый параметр по x86-64 Windows calling convention)
# Выход: RAX = результат (unsigned long long)
# ===================================================================
	.p2align 4,,15
	.globl	_Z9factoriali               # Экспортируем символ (mangled name)
	.def	_Z9factoriali;	.scl	2;	.type	32;	.endef
	.seh_proc	_Z9factoriali
_Z9factoriali:
.LFB1894:
	.seh_endprologue
	
	# === ПРОЛОГ ФУНКЦИИ: даже при O2 минимальный ===
	# (без сохранения регистров - легковесная функция)
	
	# === ПРОВЕРКА БАЗОВОГО СЛУЧАЯ ===
	cmpl	$1, %ecx                     # Сравниваем n (ECX) с 1
	jle	.L6                            # Если n <= 1, прыгаем на возврат 1
	
	# === ИНИЦИАЛИЗАЦИЯ ЦИКЛА ===
	subl	$2, %ecx                     # ECX = n - 2 (счетчик итераций)
	movl	$2, %edx                     # EDX = начальное значение i = 2
	movl	$1, %eax                     # EAX = начальный result = 1 (младшие биты)
	addq	$3, %rcx                     # RCX = (n-2) + 3 = n + 1 (условие остановки)
	
	# ВАЖНО: здесь RCX используется как верхняя граница для сравнения
	# Цикл выполняется от i=2 до i=n включительно
	
	# === ТЕЛО ЦИКЛА (с оптимизацией от компилятора) ===
	.p2align 4,,10                      # Выравнивание для производительности
.L5:
	imulq	%rdx, %rax                   # RAX = RAX * RDX (умножение)
	                                     # imulq - знаковое умножение 64-битных чисел
	addq	$1, %rdx                     # RDX++ (инкремент счетчика i)
	cmpq	%rcx, %rdx                   # Сравниваем i с (n+1)
	jne	.L5                            # Если не равно (i <= n), продолжаем цикл
	
	# === ВЫХОД ИЗ ФУНКЦИИ ===
	ret                                # Возврат (RAX уже содержит результат)
	
	# === БАЗОВЫЙ СЛУЧАЙ: возврат 1 ===
	.p2align 4,,10
.L6:
	movl	$1, %eax                     # RAX = 1
	ret                                # Выход
	.seh_endproc

# ===================================================================
# ВСПОМОГАТЕЛЬНЫЕ СИМВОЛЫ И ДАННЫЕ
# ===================================================================
	.def	__main;	.scl	2;	.type	32;	.endef  # Инициализация MinGW
	
# === СЕКЦИЯ ЧИТАЕМЫХ ДАННЫХ (строковые константы) ===
	.section .rdata,"dr"
.LC0:
	.ascii "Enter number (0-20): \0"      # Строка приглашения
	.align 8
.LC1:
	.ascii "Error: number must be between 0 and 20\12\0"  # Сообщение об ошибке
.LC2:
	.ascii "Factorial of \0"              # Для вывода результата
.LC3:
	.ascii " = \0"                        # Разделитель
.LC4:
	.ascii "\12\0"                        # Символ новой строки
.LC5:
	.ascii "Time: \0"                     # Метка времени
.LC6:
	.ascii " microseconds\12\0"          # Единицы измерения

# === ОСНОВНАЯ ФУНКЦИЯ main ===
	.section	.text.startup,"x"        # Секция startup для main
	.p2align 4,,15
	.globl	main
	.def	main;	.scl	2;	.type	32;	.endef
	.seh_proc	main
main:
.LFB1895:
	# === СОХРАНЕНИЕ РЕГИСТРОВ (Windows x64 calling convention) ===
	pushq	%rdi                         # Сохраняем RDI
	.seh_pushreg	%rdi
	pushq	%rsi                          # Сохраняем RSI
	.seh_pushreg	%rsi
	pushq	%rbx                          # Сохраняем RBX
	.seh_pushreg	%rbx
	subq	$48, %rsp                     # Выделяем 48 байт в стеке
	.seh_stackalloc	48
	.seh_endprologue
	
	# === ИНИЦИАЛИЗАЦИЯ MinGW ===
	call	__main                         # Вызов конструкторов глобальных объектов
	
	# === ВВОД ЧИСЛА ===
	# cout << "Enter number (0-20): "
	movq	.refptr._ZSt4cout(%rip), %rcx  # RCX = cout
	leaq	.LC0(%rip), %rdx               # RDX = адрес строки
	call	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
	                                   # Вызов operator<<(cout, str)
	
	# cin >> number
	movq	.refptr._ZSt3cin(%rip), %rcx   # RCX = cin
	leaq	44(%rsp), %rdx                 # RDX = адрес number (в стеке)
	call	_ZNSirsERi                     # Вызов operator>>(cin, &number)
	
	# === ПРОВЕРКА ДИАПАЗОНА ===
	cmpl	$20, 44(%rsp)                  # Сравниваем number с 20
	ja	.L15                            # Если > 20, ошибка (unsigned выше)
	
	# === ИЗМЕРЕНИЕ ВРЕМЕНИ (начало) ===
	call	_ZNSt6chrono3_V212system_clock3nowEv  # Получить текущее время
	movq	%rax, %rsi                     # RSI = start_time
	
	# === ВЫЧИСЛЕНИЕ ФАКТОРИАЛА (вставленный код) ===
	movl	44(%rsp), %eax                 # EAX = number
	cmpl	$1, %eax                       # Сравниваем с 1
	jle	.L13                            # Если <= 1, result = 1
	
	# === ОПТИМИЗИРОВАННЫЙ ЦИКЛ В main (встроенный код) ===
	# Обратите внимание: компилятор встроил функцию factorial прямо в main!
	leal	-2(%rax), %edx                # EDX = number - 2
	movl	$1, %ebx                      # EBX = result = 1
	movl	$2, %eax                      # EAX = i = 2
	addq	$3, %rdx                      # RDX = (number-2)+3 = number+1
	
	.p2align 4,,10
.L12:
	imulq	%rax, %rbx                    # result *= i
	addq	$1, %rax                      # i++
	cmpq	%rdx, %rax                    # Сравниваем i с (number+1)
	jne	.L12                           # Если не равно, продолжаем
	
.L11:
	# === ИЗМЕРЕНИЕ ВРЕМЕНИ (конец) ===
	call	_ZNSt6chrono3_V212system_clock3nowEv  # Получить текущее время
	movl	$1000, %ecx                   # Делитель для перевода в микросекунды
	movl	$13, %r8d                     # Магическое число (неиспользуемое?)
	subq	%rsi, %rax                    # RAX = end_time - start_time
	cqto                                   # Расширение знака для деления
	idivq	%rcx                          # RAX = разность в микросекундах
	
	# === ВЫВОД РЕЗУЛЬТАТА ===
	# cout << "Factorial of "
	movq	%rax, %rsi                    # Сохраняем время
	movq	.refptr._ZSt4cout(%rip), %rcx
	leaq	.LC2(%rip), %rdx
	call	_ZSt16__ostream_insertIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_PKS3_x
	
	# cout << number
	movl	44(%rsp), %edx                # EDX = number
	movq	.refptr._ZSt4cout(%rip), %rcx
	call	_ZNSolsEi                     # Вызов operator<<(cout, int)
	
	# cout << " = "
	movl	$3, %r8d                      # Длина строки = 3
	leaq	.LC3(%rip), %rdx              # " = "
	movq	%rax, %rdi                    # Сохраняем cout (возврат от предыдущего)
	movq	%rax, %rcx
	call	_ZSt16__ostream_insertIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_PKS3_x
	
	# cout << result
	movq	%rbx, %rdx                    # RDX = result (unsigned long long)
	movq	%rdi, %rcx                    # RCX = cout
	call	_ZNSo9_M_insertIyEERSoT_      # Вставка unsigned long long
	
	# cout << "\n"
	leaq	.LC4(%rip), %rdx
	movq	%rax, %rcx
	call	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
	
	# === ВЫВОД ВРЕМЕНИ ===
	# cout << "Time: "
	movq	.refptr._ZSt4cout(%rip), %rcx
	movl	$6, %r8d                      # Длина строки "Time: "
	leaq	.LC5(%rip), %rdx
	call	_ZSt16__ostream_insertIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_PKS3_x
	
	# cout << duration
	movq	.refptr._ZSt4cout(%rip), %rcx
	movq	%rsi, %rdx                    # RDX = время в микросекундах
	call	_ZNSo9_M_insertIxEERSoT_      # Вставка long long
	
	# cout << " microseconds\n"
	leaq	.LC6(%rip), %rdx
	movq	%rax, %rcx
	call	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
	
	# === УСПЕШНОЕ ЗАВЕРШЕНИЕ ===
	xorl	%eax, %eax                     # Return 0
.L8:
	addq	$48, %rsp                      # Очистка стека
	popq	%rbx                           # Восстановление RBX
	popq	%rsi                           # Восстановление RSI
	popq	%rdi                           # Восстановление RDI
	ret                                    # Выход

# === ОБРАБОТКА ОШИБКИ ===
.L15:
	movq	.refptr._ZSt4cerr(%rip), %rcx  # cerr вместо cout
	leaq	.LC1(%rip), %rdx               # Сообщение об ошибке
	call	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
	movl	$1, %eax                       # Return 1
	jmp	.L8

# === БАЗОВЫЙ СЛУЧАЙ FACTORIAL (при n<=1) ===
.L13:
	movl	$1, %ebx                       # result = 1
	jmp	.L11                           # Переход к выводу
	.seh_endproc

# ===================================================================
# ГЛОБАЛЬНАЯ ИНИЦИАЛИЗАЦИЯ C++ (для std::cout, std::cin и т.д.)
# Вызывается до main
# ===================================================================
	.p2align 4,,15
	.def	_GLOBAL__sub_I__Z9factoriali;	.scl	3;	.type	32;	.endef
	.seh_proc	_GLOBAL__sub_I__Z9factoriali
_GLOBAL__sub_I__Z9factoriali:
.LFB2411:
	subq	$40, %rsp                      # Выделение места в стеке
	.seh_stackalloc	40
	.seh_endprologue
	leaq	_ZStL8__ioinit(%rip), %rcx      # Адрес статического ioinit
	call	_ZNSt8ios_base4InitC1Ev        # Вызов конструктора
	leaq	__tcf_0(%rip), %rcx            # Адрес деструктора
	addq	$40, %rsp                      # Очистка стека
	jmp	atexit                           # Регистрация деструктора
	.seh_endproc

# == СЕКЦИЯ КОНСТРУКТОРОВ (вызов инициализации до main) ===
	.section	.ctors,"w"
	.align 8
	.quad	_GLOBAL__sub_I__Z9factoriali

# === НЕИНИЦИАЛИЗИРОВАННЫЕ ДАННЫЕ ===
.lcomm _ZStL8__ioinit,1,1               # Статический объект ioinit

# === ИНФОРМАЦИЯ О КОМПИЛЯТОРЕ ===
	.ident	"GCC: (x86_64-posix-seh-rev0, Built by MinGW-W64 project) 8.1.0"

# === ССЫЛКИ НА ВНЕШНИЕ СИМВОЛЫ ===
	.def	_ZNSt8ios_base4InitD1Ev;	.scl	2;	.type	32;	.endef
	.def	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc;	.scl	2;	.type	32;	.endef
	.def	_ZNSirsERi;	.scl	2;	.type	32;	.endef
	.def	_ZNSt6chrono3_V212system_clock3nowEv;	.scl	2;	.type	32;	.endef
	.def	_ZSt16__ostream_insertIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_PKS3_x;	.scl	2;	.type	32;	.endef
	.def	_ZNSolsEi;	.scl	2;	.type	32;	.endef
	.def	_ZNSo9_M_insertIyEERSoT_;	.scl	2;	.type	32;	.endef
	.def	_ZNSo9_M_insertIxEERSoT_;	.scl	2;	.type	32;	.endef
	.def	_ZNSt8ios_base4InitC1Ev;	.scl	2;	.type	32;	.endef
	.def	atexit;	.scl	2;	.type	32;	.endef

# === СЕКЦИИ ДЛЯ ВНЕШНИХ ССЫЛОК (cout, cin, cerr) ===
	.section	.rdata$.refptr._ZSt4cerr, "dr"
	.globl	.refptr._ZSt4cerr
	.linkonce	discard
.refptr._ZSt4cerr:
	.quad	_ZSt4cerr

	.section	.rdata$.refptr._ZSt3cin, "dr"
	.globl	.refptr._ZSt3cin
	.linkonce	discard
.refptr._ZSt3cin:
	.quad	_ZSt3cin

	.section	.rdata$.refptr._ZSt4cout, "dr"
	.globl	.refptr._ZSt4cout
	.linkonce	discard
.refptr._ZSt4cout:
	.quad	_ZSt4cout
#p.s. Надеюсь все правильно :)
</pre>

###Модульная версия программы
factorial.h
<pre>
#ifndef FACTORIAL_H
#define FACTORIAL_H

unsigned long long factorial(int n);
void writeResultToFile(int number, unsigned long long result, const char* filename);
int readNumberFromFile(const char* filename);

#endif

</pre>

factorial.cpp(Модуль)
<pre>
#include "factorial.h"
#include <fstream>
#include <iostream>

unsigned long long factorial(int n) {
    unsigned long long result = 1;
    for (int i = 2; i <= n; ++i) {
        result *= i;
    }
    return result;
}

void writeResultToFile(int number, unsigned long long result, const char* filename) {
    std::ofstream file(filename, std::ios::app);
    if (file.is_open()) {
        file << "Factorial of " << number << " = " << result << "\n";
        file.close();
    }
}

int readNumberFromFile(const char* filename) {
    std::ifstream file(filename);
    int number;
    if (file.is_open()) {
        file >> number;
        file.close();
        return number;
    }
    return -1;
}

</pre>
