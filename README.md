<p align="center">
  <img src="G5-Java-Thread-Visibility-Java.png" alt="G5-Java-Thread-Visibility-Java" width="100%">
</p>

# G5-Java-Thread-Visibility-Java
A practical Java concurrency example demonstrating why a variable updated by one thread may not be immediately visible to another thread and how `volatile` solves the visibility problem.

## Overview

In concurrent applications, updating a shared variable does not automatically guarantee that other threads will observe the new value.

Each thread may work with cached values, while the runtime and processor can optimize repeated memory operations. Because of this, code that appears correct may behave unexpectedly under concurrent execution.

This project demonstrates:

* Thread visibility problems.
* Shared mutable state.
* The difference between updating a variable and making that update visible.
* The role of the Java Memory Model.
* The use of `volatile` as a visibility guarantee between threads.

## The Problem

The example uses a worker thread controlled by a boolean flag:

```java
private boolean running = true;
```

The worker executes while the flag remains `true`:

```java
while (running) {
    counter++;
}
```

Another thread attempts to stop it:

```java
running = false;
```

At first glance, the worker should immediately observe the new value and terminate.

However, there is no explicit visibility guarantee between the thread writing `running` and the thread reading it.

As a result, the worker may continue executing using an obsolete value.

## Problem Example

```java
public class VisibilityProblem {

    static class Worker {

        private boolean running = true;

        public void runWork() {

            System.out.println("Worker iniciado...");

            long counter = 0;

            while (running) {

                counter++;

                if (counter % 1_000_000_000 == 0) {
                    // Minimal observable activity
                }
            }

            System.out.println(
                "Worker detenido. Contador: " + counter
            );
        }

        public void stop() {

            System.out.println(
                "Main solicitó detener el worker..."
            );

            running = false;
        }
    }

    public static void main(String[] args)
            throws InterruptedException {

        Worker worker = new Worker();

        Thread workerThread =
                new Thread(
                    worker::runWork,
                    "Worker-Thread"
                );

        workerThread.start();

        Thread.sleep(1000);

        worker.stop();

        workerThread.join(3000);

        if (workerThread.isAlive()) {

            System.out.println(
                "El worker sigue vivo: puede no haber visto " +
                "el cambio de running."
            );

        } else {

            System.out.println(
                "El worker terminó correctamente."
            );
        }
    }
}
```

## Why Does It Happen?

The critical instruction is:

```java
while (running)
```

One thread continuously reads `running`, while another thread modifies it.

Without an explicit memory visibility guarantee, the worker is not guaranteed to observe the updated value when expected.

The application may therefore:

* Terminate normally.
* Continue executing longer than expected.
* Keep the worker alive because it continues observing an obsolete value.

This type of problem can be particularly difficult to detect because the code may work correctly during development and fail only under particular runtime conditions.

## The Solution

The solution is to declare the shared flag as `volatile`:

```java
private volatile boolean running = true;
```

`volatile` establishes the required visibility semantics for this shared variable.

When one thread executes:

```java
running = false;
```

threads subsequently reading `running` must observe the updated value according to Java's memory visibility rules.

## Solution Example

```java
public class VisibilitySolution {

    static class Worker {

        private volatile boolean running = true;

        public void runWork() {

            System.out.println("Worker iniciado...");

            long counter = 0;

            while (running) {

                counter++;

                if (counter % 1_000_000_000 == 0) {
                    // Minimal observable activity
                }
            }

            System.out.println(
                "Worker detenido. Contador: " + counter
            );
        }

        public void stop() {

            System.out.println(
                "Main solicitó detener el worker..."
            );

            running = false;
        }
    }

    public static void main(String[] args)
            throws InterruptedException {

        Worker worker = new Worker();

        Thread workerThread =
                new Thread(
                    worker::runWork,
                    "Worker-Thread"
                );

        workerThread.start();

        Thread.sleep(1000);

        worker.stop();

        workerThread.join(3000);

        if (workerThread.isAlive()) {

            System.out.println(
                "El worker sigue vivo."
            );

        } else {

            System.out.println(
                "El worker terminó correctamente."
            );
        }
    }
}
```

## Key Concept

The important distinction is:

**Visibility**

> Can another thread observe the value that was written?

**Atomicity**

> Does an operation execute completely as one logical unit, or can other operations interfere with it?

`volatile` solves the visibility problem demonstrated in this project.

It should not be interpreted as a general replacement for synchronization when several operations must execute atomically.

## Expected Behavior

### Without `volatile`

The main thread requests the worker to stop:

```text
Worker iniciado...
Main solicitó detener el worker...
```

Depending on the execution, the worker may fail to observe the updated flag in the expected time.

The application can therefore report:

```text
El worker sigue vivo: puede no haber visto el cambio de running.
```

### With `volatile`

The worker observes the change and terminates:

```text
Worker iniciado...
Main solicitó detener el worker...
Worker detenido. Contador: ...
El worker terminó correctamente.
```

## Project Structure

```text
Java-Thread-Visibility-Demo/
│
├── src/
│   ├── VisibilityProblem.java
│   └── VisibilitySolution.java
│
├── README.md
└── LICENSE
```

## Running the Examples

Compile the problem example:

```bash
javac src/VisibilityProblem.java
```

Run it:

```bash
java -cp src VisibilityProblem
```

Compile the corrected version:

```bash
javac src/VisibilitySolution.java
```

Run it:

```bash
java -cp src VisibilitySolution
```

## Learning Objectives

After reviewing this project, a developer should understand:

* Why shared variables can cause concurrency problems.
* Why one thread changing a variable does not automatically imply immediate visibility to another thread.
* How a worker thread can continue using an obsolete value.
* What `volatile` means in this scenario.
* The difference between visibility and atomicity.
* Why concurrency errors can appear only under particular execution conditions or production workloads.

## Main Takeaway

> Updating shared state and safely communicating that update between threads are not the same thing.

When a simple variable is being used as a signal between threads, an explicit visibility guarantee is required.

In this example, that guarantee is provided by:

```java
volatile
```

## Topics

`java` `concurrency` `multithreading` `volatile` `thread-safety` `memory-visibility` `java-memory-model`

## Autor

**Work Order IT**  
Soluciones tecnológicas, arquitectura de software y formación técnica para equipos de desarrollo.

Este repositorio forma parte de una iniciativa educativa orientada a explicar cómo la concurrencia en **Python 3.13** puede acelerar un sistema o volverlo impredecible cuando el estado compartido no se controla correctamente.

Website: [www.workorder-it.net](https://www.workorder-it.net)
