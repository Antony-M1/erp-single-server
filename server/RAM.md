# Summary

This section helps how to monitor and effectively use the server RAM

### system health check
This command is a **system health check** that tells you how much RAM you have left and identifies which programs are using the most of it.

* **`free -h`**: Shows your total, used, and available memory in easy-to-read units (GB/MB).
* **`ps aux --sort=-%mem`**: Lists all active processes, ranked by their memory consumption.
* **`head -10`**: Limits the list to the **top 10** biggest memory users.

In short: it's a quick way to find out why your computer might be running slow and which "memory hog" is responsible.

```
free -h && ps aux --sort=-%mem | head -10
```
