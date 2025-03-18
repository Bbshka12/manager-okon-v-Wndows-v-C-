# manager-okon-v-Wndows-v-C-

using System;
using System.Collections.Generic;
using System.IO;
using System.Runtime.InteropServices;
using System.Text;

class Program
{
    static string logFilePath = "window_manager_log.txt";

    // WinAPI функции
    delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);
    [DllImport("user32.dll")]
    static extern bool EnumWindows(EnumWindowsProc enumProc, IntPtr lParam);
    [DllImport("user32.dll")]
    static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);
    [DllImport("user32.dll")]
    static extern int GetClassName(IntPtr hWnd, StringBuilder lpClassName, int nMaxCount);
    [DllImport("user32.dll")]
    static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
    [DllImport("user32.dll")]
    static extern bool CloseWindow(IntPtr hWnd);

    const int SW_MINIMIZE = 6;
    const int SW_MAXIMIZE = 3;
    const int SW_RESTORE = 9;

    static List<(IntPtr hWnd, string title, string className)> windows = new();

    static void Main(string[] args)
    {
        Console.WriteLine("Сканирование окон...");
        EnumWindows(EnumWindowsCallback, IntPtr.Zero);

        if (windows.Count == 0)
        {
            Console.WriteLine("Нет доступных окон.");
            return;
        }

        // Фильтрация окон
        Console.WriteLine("Введите фильтр для окон (по названию или классу), или нажмите Enter для продолжения без фильтра:");
        string filter = Console.ReadLine();

        var filteredWindows = FilterWindows(filter);

        if (filteredWindows.Count == 0)
        {
            Console.WriteLine("Нет окон, подходящих под фильтр.");
            return;
        }

        Console.WriteLine("Доступные окна:");
        for (int i = 0; i < filteredWindows.Count; i++)
        {
            Console.WriteLine($"{i + 1}: {filteredWindows[i].title} ({filteredWindows[i].className})");
        }

        Console.WriteLine("\nВведите номер окна:");
        int index;
        if (!int.TryParse(Console.ReadLine(), out index) || index < 1 || index > filteredWindows.Count)
        {
            Console.WriteLine("Неверный номер окна.");
            return;
        }

        var selectedWindow = filteredWindows[index - 1];
        Console.WriteLine($"Выбрано окно: {selectedWindow.title}");
        Console.WriteLine("Выберите действие: 1 - Минимизировать, 2 - Максимизировать, 3 - Восстановить, 4 - Закрыть");

        int action;
        if (!int.TryParse(Console.ReadLine(), out action) || action < 1 || action > 4)
        {
            Console.WriteLine("Неверное действие.");
            return;
        }

        switch (action)
        {
            case 1:
                ShowWindow(selectedWindow.hWnd, SW_MINIMIZE);
                LogAction(selectedWindow.title, "Минимизировано");
                break;
            case 2:
                ShowWindow(selectedWindow.hWnd, SW_MAXIMIZE);
                LogAction(selectedWindow.title, "Максимизировано");
                break;
            case 3:
                ShowWindow(selectedWindow.hWnd, SW_RESTORE);
                LogAction(selectedWindow.title, "Восстановлено");
                break;
            case 4:
                CloseWindow(selectedWindow.hWnd);
                LogAction(selectedWindow.title, "Закрыто");
                break;
        }
    }

    // Фильтрация окон
    static List<(IntPtr hWnd, string title, string className)> FilterWindows(string filter)
    {
        if (string.IsNullOrWhiteSpace(filter))
            return new List<(IntPtr, string, string)>(windows);

        var filteredWindows = new List<(IntPtr hWnd, string title, string className)>();
        foreach (var window in windows)
        {
            if (window.title.Contains(filter, StringComparison.OrdinalIgnoreCase) || window.className.Contains(filter, StringComparison.OrdinalIgnoreCase))
            {
                filteredWindows.Add(window);
            }
        }
        return filteredWindows;
    }

    // Обработчик окон
    static bool EnumWindowsCallback(IntPtr hWnd, IntPtr lParam)
    {
        StringBuilder title = new StringBuilder(256);
        GetWindowText(hWnd, title, title.Capacity);

        StringBuilder className = new StringBuilder(256);
        GetClassName(hWnd, className, className.Capacity);

        if (!string.IsNullOrEmpty(title.ToString()))
        {
            windows.Add((hWnd, title.ToString(), className.ToString()));
        }

        return true;
    }

    // Логирование действий
    static void LogAction(string windowTitle, string action)
    {
        string logMessage = $"[{DateTime.Now}] Окно: {windowTitle}, Действие: {action}";
        File.AppendAllText(logFilePath, logMessage + Environment.NewLine);
        Console.WriteLine(logMessage);
    }
}
