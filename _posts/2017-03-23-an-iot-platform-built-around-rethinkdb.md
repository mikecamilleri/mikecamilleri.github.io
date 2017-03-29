---
title: "An IoT Platform Built Around RethinkDB"
date: 2017-03-23
license: cc-by-sa
---

I've written before about my desire to build a home automation platform. In a [previous post](http://mikecamilleri.com/blog/home-platform/) I described my goals. More than just "home automation," I want to build a general purpose platform for collecting data, controlling devices, and making rule-based decisions. After writing that post, I wrote a preliminary specification for a platform called [RESTful Home](https://github.com/mikecamilleri/restful-home) based around RESTful HTTP APIs. REST, although an awesome paradigm for building APIs on the web, isn't an ideal solution for communicating between microservices.

After deciding to move away from HTTP REST APIs, I was going to build the system around a message broker such as [RabbitMQ](http://www.rabbitmq.com), [NATS](http://nats.io), or [Apache Kafka](https://kafka.apache.org) paired with a database (or two). While certainly a good way to architect this system, during design, I realized that there would be a lot of complexity involved, particularly in allowing the various parts of the system to communicate with the database via the message broker. Although doable, I discovered a better solution, [RethinkDB](https://rethinkdb.com). 

RethinkDB is non-relational database with a unique (and exceptionally useful) feature. In addition to polling for changes as in a typical database, an application can subscribe to a query and RethinkDB will push new results to the application in real time. This is a feature they call [changefeeds](https://rethinkdb.com/docs/changefeeds/ruby/). Changefeeds allow RethinkDB to serve as database, message broker, and real-time repository of system state. 

## Components

The RethinkDB database **is** the core of the home automation platform and will consist of four tables: `modules`, `gateways`, `devices`, and `values`. The first three of those represent the three major types of components that make up the system. The term "component" refers to a discrete software application (microservice). Multiple components may share hardware or even reside in the cloud. 

### Modules

Modules may perform the automation functions typically associated with IoT applications and also may provide interfaces for the user to interact with the system. The set of basic modules might include `web-admin` and `automaton`. Additionally there will be a module, probably called `core`, which will perform setup (e.g. reading configuration files) and housekeeping work. Each module will be represented in the database by a document in the `modules` table. Modules will listen for changes to their document via a changefeed and change settings on other components by writing to their documents. Voila! RethinkDB is now a pub-sub message broker!

```json
{
	"id": "random-id-for-this-module",
	"name": "module-name",
	"prettyName": "Module Name",
	"webAdminURL": "http://0.0.0.0:80/web-admin-interface-url-if-available",
	"config": {
		"arbitraryKey": "arbitrary-value-of-arbitrary-type"
	},
	"status": {
		"arbitraryKey": "arbitrary-value-of-arbitrary-type"
	}
}
```

### Gateways

Gateways serve as intermediaries between devices and the core database. Gateways might include one for Z-Wave devices, a web scraper, and a home brew alarm panel. Gateways are represented in the database by a documents in the `gateways` table. Like modules, gateways listen for changes to the document representing themselves. Additionally gateways listen to changes to their devices' documents and write to them and the `values` table on their behalf.  

```json
{
	"id": "random-id-for-this-gateway",
	"name": "gateway-name",
	"prettyName": "Gateway Name",
	"webAdminURL": "http://0.0.0.0:80/web-admin-interface-url-if-available",
	"config": {
		"arbitraryKey": "arbitrary-value-of-arbitrary-type",
	},
	"status": {
		"arbitraryKey": "arbitrary-value-of-arbitrary-type"
	}
}
```

### Devices

Devices are things that the platform receives data from or controls. Devices may be physical such as thermostats, light switches, or alarm sensors; or they may be virtual such as Twitter feed or 3rd party weather API. Devices will be represented by a document in the `devices` table.

```json
{
	"id": "random-id-for-this-device",
	"name": "device-name",
	"prettyName": "Device Name",
	"webAdminURL": "http://0.0.0.0:80/web-admin-interface-url-if-available",
	"gatewayID": "id-of-the-gateway-this-device-is-attached-to",
	"features": [
		{
			"name": "name-of-feature",
			"prettyName": "Name of Feature",
			"type": "from-predefined-list-of-supported-types",
			"setting": "value-appropriate-for-type"
		}
	],
	"config": {
		"arbitraryKey": "arbitrary-value-of-arbitrary-type",
		"refreshIntervalSeconds": 60
	},
	"status": {
		"arbitraryKey": "arbitrary-value-of-arbitrary-type"
	}
}
```

### Values

Some devices collect date over time that my be interesting to analyze or be useful for predictive automation or AI. Gateways will write this data to the `values` table.

```json
{
	"id": "random-id-for-this-value",
	"deviceID": "id-of-the-device-that-wrote-this-value",
	"time": "RethinkDB time type",
	"type": "from-predefined-list-of-supported-types",
	"value": "value-appropriate-for-type"
}
```

## Examples

That's a lot of information, so here are a few examples illustrating how this all works. 

### Using a web interface (`web-admin` module), a user changes the setting of _Downstairs Thermostat_ to 65°F.

1. The user accesses the appropriate page in the web interface.
2. The `web-admin` module reads the `thermostat-downstairs` document in the `devices` table to present the current setting to the user.
3. The user changes the setting using the web interface.
4. The `web-admin` module writes the update to the `thermostat-downstairs` document.
5. The appropriate gateway, which has a changefeed listening to the `thermostat-downstairs` document, updates the thermostat. 

### A homeowner is interested in recording exterior temperature.

1. The homeowner, through one of the modules that provide an admin interface, configures an appropriate feature on an appropriate device to take temperature readings at some interval. This works similarly to the example above
2. The gateway that the device is attached to writes the temperature values to the `values` table at the specified intervals. 

### A homeowner wants some devices in his home to enter a certain state when an alarm is triggered. 

1. An alarm is triggered on some device.
2. The gateway that device is attached to writes the state change to the device's document.
3. The `automation` module receives notice of this state change on a changefeed.
4. The `automation` module writes to the documents representing several devices, changing their settings as needed.
5. The appropriate gateways are notified of the change via their changefeeds.
5. Lights turn on, a siren sounds, a text message is sent to the homeowner, and so on. 

### Someone turns on a Z-Wave enabled light switch

1. The light switch closes the circuit turning on the light, just like a regular light switch.
2. The light switch sends a Z-Wave message to the Z-Wave gateway.
3. The gateway updates the document representing that switch.
4. Modules see the change on their changefeed and can perform appropriate actions such as updating an admin interface or taking automation action. 

## A few more things

One huge advantage of a microservices architecture is the ability to build the various services using the tools most appropriate for the job. This will be especially useful when building the gateways using existing libraries. The [major open source library for the Z-Wave protocol](https://github.com/OpenZWave/open-zwave) is written in C++, for example. I don't want to build this whole system in C++ so the ability to isolate that part in a microservice is great. Another positive is licensing. The complete separation of components allows each component to be licensed separately. This is useful, for example, if I want to license most of my own code under a permissive license like the MIT or Apache licenses, but use GPL code in certain components. And, of course, microservices can live on one, or multiple machines as needed. I would like to run all of this, including the database, on a few Raspberry Pi single board computers. 
