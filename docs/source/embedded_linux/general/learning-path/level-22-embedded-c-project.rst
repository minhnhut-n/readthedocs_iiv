Level 22 – Embedded C Project (Design Patterns & RTOS)
======================================================

Project Overview
----------------

Build one project: a modular environmental-monitoring and control node.

It can run first on your PC with a fake RTOS/HAL, then on an STM32, ESP32, or Zephyr
board. It reads temperature/light/motion, displays/logs values, triggers alarms, and
supports interchangeable drivers. That naturally covers nearly every pattern in the
course.

C Design Patterns & Application Matrix
--------------------------------------

- Object: sensor_t, logger_t, alarm_t objects with init/read/deinit
- Opaque: Hide each driver's private state in .c; expose handles in .h
- Singleton: One system logger and one scheduler/config service
- Factory: Create sensor instances from configuration: I2C temp sensor, fake sensor, ADC light sensor
- Callback: Notify display, logger, and alarm when a new sample arrives
- Inheritance: Base sensor_t; derived temperature, light, and motion sensors
- Virtual API: Same sensor_read() API for real and simulated sensors
- Bridge: Separate display abstraction from OLED/LCD/console implementations
- Return values: Consistent 0 success, negative errno failures, outputs by pointer
- Spinlock: Protect ISR-shared ring-buffer indices or sample flags
- Semaphore: ISR signals a sampling/processing task that data is ready
- Mutex: Serialize I2C bus, UART logger, or configuration access
- Condition variable: Logger task waits until buffer has data; command task waits for a configuration change

Best Learning Sequence
----------------------

1. Week 1 — Single-threaded core:
   Implement object, opaque, return-value, and factory patterns. Use fake sensors so debugging stays easy.

2. Week 2 — Extensibility:
   Add base sensor + two concrete sensors, virtual API, callbacks, and console/OLED display bridge.
   Key test: Add a new sensor or display without changing application logic.

3. Week 3 — RTOS behavior:
   Add three tasks: sampler, processor, logger. Use semaphore, mutex, condition variable, then an ISR-safe ring buffer guarded by interrupt masking/spinlock.

4. Week 4 — Make it credible:
   Add fault injection: disconnected sensor, full log buffer, slow display, simultaneous I2C access. Write a short README explaining which pattern solves each problem.

Target Architecture
-------------------

.. code-block:: text

   Sensor Factory -> Sensor API -> Callback subscribers
       |                             ├── Alarm
       |                             ├── Display bridge
       v                             └── Logger singleton
   RTOS queues/semaphores/mutexes

Key Takeaway
------------

Avoid starting with real hardware and every pattern at once. First make a desktop
simulation produce values every second; then replace one component at a time with
hardware drivers. You'll learn the patterns because each one removes a concrete pain
point.

.. include:: ../../../_includes/contact_info.rst