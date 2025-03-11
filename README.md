# SyncAsyncParallel
using System;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

public class Practos2
{ 
    static string directoryPath = "C:\\Files";
    static string[] files = { "file_1.txt", "file_2.txt", "file_3.txt", "file_4.txt", "file_5.txt" }; 

    public static void Main(string[] args)
    {
        Console.WriteLine($"Sync: {Time(() => CountCharsSync()).chars} символов, {Time(() => CountCharsSync()).time}ms");
        Console.WriteLine($"Async: {Time(() => CountCharsAsync().Result).chars} символов, {Time(() => CountCharsAsync().Result).time}ms");
        Console.WriteLine($"Parallel: {Time(() => CountCharsParallel()).chars} символов, {Time(() => CountCharsParallel()).time}ms");
    }

    static (long chars, long time) Time(Func<long> action)
    {
        Stopwatch sw = Stopwatch.StartNew();
        long chars = action();
        sw.Stop();
        return (chars, sw.ElapsedMilliseconds);
    }

    static long CountCharsSync()
    {
        long total = 0;
        foreach (string file in files)
        {
            total += File.ReadAllText(Path.Combine(directoryPath, file)).Length;
        }
        return total;
    }

    static async Task<long> CountCharsAsync()
    {
        long total = 0;
        foreach (string file in files)
        {
            total += (await File.ReadAllTextAsync(Path.Combine(directoryPath, file))).Length;
        }
        return total;
    }

    static long CountCharsParallel()
    {
        long total = 0;
        Parallel.ForEach(files, file =>
        {
            Interlocked.Add(ref total, File.ReadAllText(Path.Combine(directoryPath, file)).Length);
        });
        return total;
    }
}

