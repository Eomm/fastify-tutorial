# Plugins

Let's start the discussion around plugins. They are the main building blocks of a Fastify application. They help encapsulate and share functionality across the same application or multiple ones. The main rule of a plugin is that everything declared inside a plugin is relative to the plugin itself and it's children. The only way to escape this rule is to decorate your plugin using the package `fastify-plugin`.

Let's test this behavior so we can understand it better.
