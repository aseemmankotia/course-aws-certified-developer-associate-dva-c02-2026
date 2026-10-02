# Chapter 3 Lab: Amazon DynamoDB: Key Design, GSIs/LSIs, Streams, and DAX

**Estimated time:** 25 minutes

## Exercise

Design a DynamoDB partition key and GSI for an e-commerce order table (CustomerID as partition key, OrderDate as sort key, GSI on OrderStatus), then read a boto3 conditional write and a DynamoDB Streams-triggered Lambda handler

## Key takeaway

DynamoDB exam questions are key-design and index-selection problems in disguise — always map the access pattern to the correct key and index type first.
