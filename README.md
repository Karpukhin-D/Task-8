# Рубежный практический контроль: Комплексный практикум по базовым конструкциям C#
# Вариант 20. Контроллер испытательного вибростенда деталей
## Выполнил студент группы П25-2.1 Карпухин Дмитрий
---
<picture>
  <img src="https://github.com/Karpukhin-D/Task-8/blob/main/Рубежный%20контроль/1.png">
</picture>


<picture>
  <img src="https://github.com/Karpukhin-D/Task-8/blob/main/Рубежный%20контроль/2.png">
</picture>

```csharp

using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== КОНТРОЛЛЕР ИСПЫТАТЕЛЬНОГО ВИБРОСТЕНДА ===\n");

        // Задание 1. Побитовая упаковка
        int frequency = 500;
        int amplitude = 125;
        int cycles = 1200;
        int crackStatus = 2;

        uint packed = (uint)frequency | //сдвигаем каждое значение на место его битового поля
                      ((uint)amplitude << 10) |
                      ((uint)cycles << 18) |
                      ((uint)crackStatus << 30);

        Console.WriteLine("Задание 1. Побитовая упаковка");
        Console.WriteLine($"Упакованное значение: {packed}");

        int unpackedFrequency = (int)(packed & ((1u << 10) - 1)); //распаковка частоты: 1u — единица типа uint, << 10 сдвигает единицу на 10 бит влево,
                                                                  //вычитаем 1, чтобы получить 10 единичных битов
        int unpackedAmplitude = (int)((packed >> 10) & ((1u << 8) - 1)); //значение амплитуды
        int unpackedCycles = (int)((packed >> 18) & ((1u << 12) - 1)); //число циклов
        int unpackedCrackStatus = (int)((packed >> 30) & ((1u << 2) - 1)); //статус трещины

        Console.WriteLine($"Частота: {unpackedFrequency} Гц");
        Console.WriteLine($"Амплитуда: {unpackedAmplitude} сотых мм");
        Console.WriteLine($"Число циклов: {unpackedCycles} сотен тысяч");
        Console.WriteLine($"Статус трещины: {unpackedCrackStatus}");

        uint parityValue = packed; //подсчет количества единичных битов в packed
        int ones = 0;

        while (parityValue != 0) 
        {
            ones += (int)(parityValue & 1u); //& 1u проверяет младший бит числа (т. е. самый правый бит)
            parityValue >>= 1;
        }

        bool evenParity = (ones & 1) == 0;
        Console.WriteLine($"Проверка четности: {evenParity}\n");

        // Задание 2. Матрица напряжений размером 6 на 6
        int[,] stressMatrix =
        {
            { 10, 20, 30, 40, 50, 60 },
            { 12, 24, 36, 48, 60, 72 },
            { 15, 30, 45, 60, 75, 90 },
            { 14, 28, 42, 56, 70, 84 },
            { 11, 22, 33, 44, 55, 66 },
            { 16, 32, 48, 64, 80, 96 }
        };

        int rows = stressMatrix.GetLength(0);
        int cols = stressMatrix.GetLength(1);

        Console.WriteLine("Задание 2. Матрица напряжений");

        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
                Console.Write($"{stressMatrix[i, j],4}");
            Console.WriteLine();
        }

        for (int i = 0; i < rows; i++)
        {
            int max = stressMatrix[i, 0]; //считаем первый элемент максимумом текущей строки

            for (int j = 1; j < cols; j++)
            {
                if (stressMatrix[i, j] > max)
                    max = stressMatrix[i, j];
            }

            for (int j = 0; j < cols; j++)
                stressMatrix[i, j] /= max;
        }

        Console.WriteLine("\nМатрица после преобразования:");

        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
                Console.Write($"{stressMatrix[i, j],4}");
            Console.WriteLine();
        }

        // Задание 3. Фильтрация вибрационных пиков
        int[] frequencies = { 100, 250, 400, 600, 800 };
        int[] amplitudes = { 20, 95, 3, 110, 70 };

        FilterVibrationPeaks(
            ref frequencies, ref amplitudes, out int dangerousPeaks,
            80, 90, 100);

        Console.WriteLine("\nЗадание 3. Фильтрация вибрационных пиков");
        Console.WriteLine($"Опасных пиков: {dangerousPeaks}");

        for (int i = 0; i < frequencies.Length; i++)
            Console.WriteLine($"Частота: {frequencies[i]} Гц, амплитуда: {amplitudes[i]}");

        // Задание 4. Рекурсия
        int n = 7;
        Console.WriteLine($"\nЗадание 4. Число Фибоначчи: F({n}) = {ComputeResonanceHarmonic(n)}");

        // Задание 5. Перегрузка первого метода
        int cyclesCompleted = 750000; //выполнено циклов
        int maxCycles = 1000000; //максимальный ресурс

        double fatiguePercent = AssessFatigueLife(cyclesCompleted, maxCycles); //версия метода с двумя параметрами int

        // Задание 5. Вторая перегрузка метода
        int[] stressCycles = { 1000, 2000, 1500 };
        int[] stressLevelsMpa = { 50, 80, 100 };

        AssessFatigueLife(stressCycles, stressLevelsMpa, out double damageAccumulated); //версия метода с двумя массивами и параметром out

        Console.WriteLine("\nЗадание 5. Оценка усталости детали");
        Console.WriteLine($"Исчерпание ресурса: {fatiguePercent:F2}%");
        Console.WriteLine($"Накопленное повреждение: {damageAccumulated:F4}");
    }

    // Задание 3. Метод фильтрации
    static void FilterVibrationPeaks(
        ref int[] frequencies, ref int[] amplitudes,
        out int dangerousPeaks, params int[] amplitudeLimits)
    {
        dangerousPeaks = 0;

        for (int i = 0; i < amplitudes.Length; i++)
        {
            if (amplitudes[i] <= 5) //значения до 5 включительно это ложные всплески
            {
                frequencies[i] = 0;
                amplitudes[i] = 0;
            }
            //есть ли ограничения и превышено ли одно из них
            else if (amplitudeLimits.Length > 0 &&
                     amplitudes[i] > amplitudeLimits[i % amplitudeLimits.Length])
            {
                dangerousPeaks++;
            }
        }
    }

    // Задание 4. Рекурсивный метод
    static int ComputeResonanceHarmonic(int n)
    {
        if (n <= 1)
            return n;

        return ComputeResonanceHarmonic(n - 1) +
               ComputeResonanceHarmonic(n - 2);
    }

    // Задание 5. Первая перегрузка
    static double AssessFatigueLife(int cyclesCompleted, int maxCycles)
    {
        if (maxCycles <= 0) //рассчитывать процент при нулевом или отрицательном максимальном ресурсе нельзя
            return 0;

        return (double)cyclesCompleted / maxCycles * 100;
    }

    // Задание 5. Вторая перегрузка
    static double AssessFatigueLife(
        int[] stressCycles, int[] stressLevelsMpa,
        out double damageAccumulated)
    {
        damageAccumulated = 0;

        if (stressCycles.Length != stressLevelsMpa.Length)
            return 0;

        //сумма вкладов всех нагрузок
        for (int i = 0; i < stressCycles.Length; i++)
        {
            damageAccumulated +=
                (double)stressCycles[i] * stressLevelsMpa[i] / 1000000;
        }

        return damageAccumulated;
    }
}
```
