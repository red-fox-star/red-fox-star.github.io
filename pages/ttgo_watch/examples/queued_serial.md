---
layout: home
title: Queued Serial Port Messages Task
---

### serial_message_queue.h

{% highlight c++ linenos %}
#pragma once

// original: https://foxpost.xyz/pages/ttgo_watch/examples/queued_serial
// updated: 2020-07-10

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "Arduino.h"

struct SerialMessage {
  char text[255];
  uint32_t time;
};

void q_message(String text);
void q_message(const char * text);
void q_message_ln(const char * text);
void q_message_fmt(const char * format, ...);
void setupSerialMessenger();

extern TaskHandle_t serial;
extern QueueHandle_t serial_queue;
{% endhighlight %}

### serial_message_queue.cpp

{% highlight c++ linenos %}
#include "serial_message_queue.h"

// original: https://foxpost.xyz/pages/ttgo_watch/examples/queued_serial
// updated: 2020-07-10

TaskHandle_t serial;
QueueHandle_t serial_queue;

void q_message(String text) { q_message(text.c_str()); }
void q_message_ln(String text) { q_message_ln(text.c_str()); }

void q_message_ln(const char * text) {
  char extended[strlen(text) + 2];
  strncpy(extended, text, sizeof(extended));
  strcat(extended, "\n");
  q_message(extended);
}

void q_message(const char * text) {
  SerialMessage message;
  message.text[0] = 0x0;
  strcpy(message.text, text);
  message.time = micros();

  xQueueSend(serial_queue, (void *) &message, 0);
}

void q_message_fmt(const char * format, ...) {
  char buffer[256];
  va_list args;
  va_start(args, format);
  vsnprintf(buffer, 256, format, args);
  va_end(args);

  q_message(buffer);
}

void setupSerialMessenger() {
  Serial.begin(115200);

  serial_queue = xQueueCreate(10, sizeof(SerialMessage));

  xTaskCreate(
    [](void* _) {
      for (;;) {
        SerialMessage message;
        auto success = xQueueReceive(serial_queue, &message, 10);
        if (success != pdPASS) continue;
        Serial.printf("%u - %s", message.time, message.text);
      }
    },
    "serial", 10000, NULL, 5, &serial
  );

  // xTaskCreate(
  //   [](void* _) {
  //     for (;;) {
  //       q_message_ln("HUP");
  //       vTaskDelay(10000 / portTICK_PERIOD_MS);
  //     }
  //   },
  //   "serial_hup_task", 10000, NULL, 30, NULL
  // );
}
{% endhighlight %}
