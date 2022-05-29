# Producers and consumers problem

![Image](/producers_consumers.png "Producers and consumers")

## Description
There are multiple producers and consumers.<br>
Each producer can push item to the queue until queue size limit is reached.<br>
Each consumer can pop item from the queue until queue is empty.<br>
Accessing queue is not allowed simultaneously: only one producer/consumer can push/pop item at a time.<br>

## Solution using mutex

For each push/pop from the queue, lock mutex for exclusive access:

```cpp
    void produce_mutex(std::queue<int> &q)
    {
        while (counter < target)
        {
            if (q.size() < queue_size_limit)
            {
                std::lock_guard<std::mutex> lock(m);
                if (q.size() < queue_size_limit)
                {
                    q.push(random());
                }
            }
        }
    }

    void consume_mutex(std::queue<int> &q)
    {
        while (counter < target)
        {
            if (!q.empty())
            {
                std::lock_guard<std::mutex> lock(m);
                if (!q.empty())
                {
                    q.pop();
                    counter++;
                }
            }
        }
    }
```
Downside is the huge amount of context switches.

## Solution using condition variable
Do push/pop iterations in a loop while is possible.<br>
Afterwards notify and unblock other threads.

```cpp
    void produce_cv(std::queue<int> &q)
    {
        std::unique_lock lk(m);
        while (counter < target)
        {
            cv.wait(lk, [&q]() { return q.size() < queue_size_limit; });
            q.push(random());
            cv.notify_all();
        }
    }

    void consume_cv(std::queue<int> &q)
    {
        std::unique_lock lk(m);
        while (counter < target)
        {
            cv.wait(lk, [&q]() { return !q.empty(); });
            q.pop();
            counter++;
            cv.notify_all();
        }
    }
```

Testing results:

```bash
Running test #1
Mutex               : workers amount = 8, elapsed time = 1260 ms
Conditional variable: workers amount = 8, elapsed time = 435 ms
Running test #2
Mutex               : workers amount = 8, elapsed time = 1281 ms
Conditional variable: workers amount = 8, elapsed time = 384 ms
Running test #3
Mutex               : workers amount = 8, elapsed time = 1298 ms
Conditional variable: workers amount = 8, elapsed time = 391 ms
Running test #4
Mutex               : workers amount = 8, elapsed time = 1321 ms
Conditional variable: workers amount = 8, elapsed time = 397 ms
Running test #5
Mutex               : workers amount = 8, elapsed time = 1339 ms
Conditional variable: workers amount = 8, elapsed time = 440 ms
```

Implementation using condition variable gives x3 speed-up due to less amount of context switches.

Full source code listing:

```cpp
#include <atomic>
#include <chrono>
#include <condition_variable>
#include <functional>
#include <iostream>
#include <mutex>
#include <queue>
#include <ratio>
#include <string>
#include <thread>

class Solution;
typedef void (Solution::*ConsumerMemFn)(std::queue<int> &q);
typedef void (Solution::*ProducerMemFn)(std::queue<int> &q);

class Solution
{
  public:
    void run()
    {
        for (int t = 0; t < 5; ++t)
        {
            printf("Running test #%d\n", t + 1);
            run_test("Mutex               ", &Solution::produce_mutex, &Solution::consume_mutex);
            run_test("Conditional variable", &Solution::produce_cv, &Solution::consume_cv);
        }
    }

    void run_test(std::string testName, ProducerMemFn producer, ConsumerMemFn consumer)
    {
        const std::size_t N = std::max(std::thread::hardware_concurrency(), 2u);
        std::queue<int> q;

        counter = 0;
        auto start = std::chrono::high_resolution_clock::now();
        std::vector<std::thread> workers;
        for (std::size_t i = 0; i < N / 2; ++i)
        {
            workers.emplace_back(producer, this, std::ref(q));
            workers.emplace_back(consumer, this, std::ref(q));
        }
        std::for_each(workers.begin(), workers.end(), std::mem_fn(&std::thread::join));

        auto stop = std::chrono::high_resolution_clock::now();
        auto duration_ms = std::chrono::duration_cast<std::chrono::milliseconds>(stop - start).count();

        printf("%s: workers amount = %zu, elapsed time = %lu ms\n", testName.c_str(), workers.size(), duration_ms);
    }

  private:
    void produce_mutex(std::queue<int> &q)
    {
        while (counter < target)
        {
            if (q.size() < queue_size_limit)
            {
                std::lock_guard<std::mutex> lock(m);
                if (q.size() < queue_size_limit)
                {
                    q.push(random());
                }
            }
        }
    }

    void consume_mutex(std::queue<int> &q)
    {
        while (counter < target)
        {
            if (!q.empty())
            {
                std::lock_guard<std::mutex> lock(m);
                if (!q.empty())
                {
                    q.pop();
                    counter++;
                }
            }
        }
    }

    void produce_cv(std::queue<int> &q)
    {
        std::unique_lock lk(m);
        while (counter < target)
        {
            cv.wait(lk, [&q]() { return q.size() < queue_size_limit; });
            q.push(random());
            cv.notify_all();
        }
    }

    void consume_cv(std::queue<int> &q)
    {
        std::unique_lock lk(m);
        while (counter < target)
        {
            cv.wait(lk, [&q]() { return !q.empty(); });
            q.pop();
            counter++;
            cv.notify_all();
        }
    }

    const size_t target = 2'000'000;
    constexpr static size_t queue_size_limit = 100;
    std::atomic<uint32_t> counter = 0;
    std::mutex m;
    std::condition_variable cv;
};

int main()
{
    Solution().run();
    return 0;
}
```

