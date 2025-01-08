# Insecure Deserialization

## Prerequisites

**Serialization** converts complex data structures such as objects and their fields into a format that can be sent and received as a sequential stream of bytes. Serialized data format can be either binary or string.  

**Deserialiaztion** restors the byte stream to a fully functional replica of the original object, in the exact state as when it was serialized. 

## What Is Insecure Deserialization?
Insecure Deserialization occurs when untrusted data is used to abuse the logic of an application's deserialialization process
