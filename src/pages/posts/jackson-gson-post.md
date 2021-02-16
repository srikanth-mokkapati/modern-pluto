---
title: "Jackson and Gson - A Handshake \U0001F91D"
date: '2021-02-14'
thumb_img_alt: Jackson and Gson - A Handshake
excerpt: >-
  Find out how to achieve a Cross-Functionality between the Two Best Java JSON
  Libraries
seo:
  title: ''
  description: ''
  robots: []
  extra: []
  type: stackbit_page_meta
template: post
thumb_img_path: images/Screenshot JSON.png
content_img_alt: JSON
subtitle: JSON Parsing in Java
content_img_path: images/Screenshot JSON.png
---
## Introduction

[Jackson](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core/2.10.2) and [Gson](https://mvnrepository.com/artifact/com.google.code.gson/gson/2.8.6) are two of the most popular Java Libraries used for JSON Parsing and API response Serialisation/De-serialisation while using Java and specifically SpringBoot. Both have their own advantages and short-comings. In most cases, it might be sufficient to use a single library. But, what if, you need to use both of them? Continue reading, to find out.

## Use cases

By default, Spring Framework uses Jackson library for parsing Rest API Input and Output. It converts object responses to JSON and request body to Object. And, With simple tinkering to Application properties we can enable Gson parsing.

However, in some cases, what if we needed the output to be JSON Objects like JSON Array or JSON Object? If we are using Gson, it has straight forward JsonObject and JsonArray Classes which can be used and be request body or response. If we are using Jackson there are no direct classes, but we have JsonNode, ArrayNode etc.. These classes are not very intuitive. If you are to use Gson classes with Jackson parsing in Request Body/Response you will face an exception such as below.

\`Could not write JSON: JsonObject; nested exception is com.fasterxml.jackson.databind.JsonMappingException\`

This can be on a myriads of internal gson.JsonObject methods/properties, as Jackson is getter/setter based, and on different one each time.

## Alternatives

*   Use Java Collections : Use Maps/Arrays/Lists to replace JSON Object/Array Usage. This may keep the code simple, But Business logic may not permit this always.

*   Use Other Compatible Libraries : We can use other JSON Libraries such as [JSON - Small and Fast Parser](https://mvnrepository.com/artifact/net.minidev/json-smart/1.0.9) or Default Package [Java JSON](https://mvnrepository.com/artifact/org.json/json). The classes  extend Java Collections, fundamentally parse and hold JSON fields as objects, and hence can be converted without any additional changes. However, there are several performance benchmarks which suggest they are not as fast. At the same time, some of us may prefer the ObjectMapper and Gson classes for Object parsing and Type Conversions and want to keep imports at minimum.

## The Solution

**Custom Serialisers/Deserialisers** are powerful extensions in both Jackson and Gson Libraries. In this example, I will present how they can be written for Gson specific objects for Jackson Parsing at API end. The vice-versa is also possible but takes a lot more work considering multiple jackson classes like IntegerNode, NullNode, etc...which extend the Jackson base JSON class TreeNode. Also, it is simpler to use Gson classes like JsonObject and JsonArray for internal business logic.

**The Classes** :

*   Gson Classes JsonObject and JsonArray are self-explanatory.

*   Nulls are denoted by JsonNull.

*   Primitives(String/Number/Boolean) by JsonPrimitive.

*   All the four classes extend JsonElement abstract class.

*   Writing Custom Serialiser for JsonElement will suffice.

*   We have to keep in mind, the JSON supported types and how they are written (quotes/no quotes).

**CustomSerialiser** :

```
public class CustomGsonObjectSerializer extends JsonSerializer<JsonElement> {
    @Override
    public void serialize(JsonElement value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        if(value.isJsonObject()) {
            writeJsonObject(value.getAsJsonObject(), gen, serializers);
        } else if (value.isJsonArray()) {
            writeJsonArray(value.getAsJsonArray(), gen, serializers);
        } else if (value.isJsonPrimitive()) {
            writeJsonPrimitive(value.getAsJsonPrimitive(), gen, serializers);
        } else if (value.isJsonNull()) {
            gen.writeNull();
        } else {
            throw new UnsupportedOperationException("Unsupported Gson - JsonElement Type.");
        }
    }

    private void writeJsonPrimitive(JsonPrimitive value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeObject(processJsonPrimitive(value));
    }

    private void writeJsonArray(JsonArray value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeStartArray();
        for (JsonElement element : value) {
            gen.writeObject(element);
        }
        gen.writeEndArray();
    }

    private void writeJsonObject(JsonObject value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        gen.writeStartObject();
        for(Map.Entry<String,JsonElement> entry : value.entrySet()) {
            gen.writeFieldName(entry.getKey());
            gen.writeObject(entry.getValue());
        }
        gen.writeEndObject();
    }

    private Object processJsonPrimitive(JsonPrimitive value) {
        if(value.isBoolean()) {
            return value.getAsBoolean();
        } else if (value.isString()) {
            return value.getAsString(); //toString method will add extra quotes
        } else if (value.isNumber()) {
            return NumberUtils.createNumber(value.getAsString()); //apache lang3 library
        } else {
            throw new UnsupportedOperationException("UnSupported Gson - JsonPrimitive Type.");
        }
    }
}

```

Notable points :

*   Code is written taking the types into account and throwing exception incase the type is not supported. With this, we can identify if Gson Library updates are not properly tracked.

*   We can also handle Number generation from String separately instead of using Apache Utils, but be careful while handling Big Integers/Decimals.

*   Null Values are written to output, while using this code, however we can tinker the if-else conditions to prevent that if needed.

**Custom Deserialiser** :

*   **I would be against this at present**, as it is only a sub-optimal solution. We can better go with Java Collections(Maps/Lists) rather than using Gson Classes to keep it simple. It is just in-case there is no other alternative.

*   We need separate Deserializers for JsonArray and JsonObject and having a String conversion in between essentially makes it that we are parsing input twice.

For JsonObject :

    public class CustomJsonObjectDeserializer extends JsonDeserializer<JsonObject> {

        @Override
        public JsonObject deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
            JsonElement jsonElement =  com.google.gson.JsonParser.parseString(p.readValueAsTree().toString());
            if (jsonElement.isJsonNull()) {
                return null;
            } else if (jsonElement.isJsonObject()) {
                return jsonElement.getAsJsonObject();
            } else {
                throw new JsonParseException(p, "Input not a JsonObject");
            }
        }

    }

For JsonArray :

    public class CustomJsonArrayDeserializer extends JsonDeserializer<JsonArray> {

        @Override
        public JsonArray deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
            JsonElement jsonElement =  com.google.gson.JsonParser.parseString(p.readValueAsTree().toString());
            if (jsonElement.isJsonNull()) {
                return null;
            } else if (jsonElement.isJsonArray()) {
                return jsonElement.getAsJsonArray();
            } else {
                throw new JsonParseException(p, "Input not a JsonArray");
            }
        }
    }

**Register the Custom Implementations** :

We need to register these custom implementations to be used while Http requests are processed. We can use simple Bean Configuration for this.

    @Configuration
    public class JacksonConfiguration {


        @Bean
        public ObjectMapper objectMapper() {
            ObjectMapper mapper = new ObjectMapper();
            SimpleModule simpleModule = new SimpleModule();
            simpleModule.addSerializer(JsonElement.class, new CustomGsonObjectSerializer());
            simpleModule.addDeserializer(JsonObject.class, new CustomJsonObjectDeserializer());
            simpleModule.addDeserializer(JsonArray.class, new CustomJsonArrayDeserializer());
            mapper.registerModule(simpleModule);
            return mapper;
        }

    }

**Testing** :

Writing a simple Rest API Controller to test this :

    @RestController
    public class SampleRestController {

        @GetMapping("/get")
        public JsonObject status() {
            JsonObject jsonObject = new JsonObject();
            jsonObject.addProperty("String", "string");
            jsonObject.addProperty("number", 1);
            jsonObject.addProperty("boolean", "true");
            //any null property added will automatically becomes JsonNull
            jsonObject.add("null", JsonNull.INSTANCE);
            JsonArray jsonArray = new JsonArray();
            jsonArray.add("string");
            jsonArray.add(1.1);
            jsonArray.add(false);
            jsonArray.add(JsonNull.INSTANCE);
            jsonObject.add("jsonArray", jsonArray);
            return jsonObject;
        }

        @PostMapping("/post")
        public JsonObject post(@RequestBody JsonObject jsonObject) {
            return jsonObject;
        }
    }

We get the following output on testing GET endpoint and giving its response as input to POST Endpoint.

    {
        "String": "string",
        "number": 1,
        "boolean": "true",
        "null": null,
        "jsonArray": [
            "string",
            1.1,
            false,
            null
        ]
    }

## Conclusion&#xD;&#xA;

Gson and Jackson are two of the most powerful and extensible libraries while handling JSON. Personally, I prefer Gson for Type conversions and internal JSON Logic and Jackson for Http Request handling.

The Java community maybe biased while choosing between these two. But, I Hope, this may help you incase cross-functionality is in-evitable.
