# Creating a Database with DynamoDB and AWS CLI

This guide will walk you through the process of creating a DynamoDB database using AWS CLI. Below are the main steps:

## Create a Table

```bash
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema \
        AttributeName=Artist,KeyType=HASH \
        AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

## Insert an Item

```bash
aws dynamodb put-item \
    --table-name Music \
    --item file://itemmusic.json
```

## Insert Multiple Items

```bash
aws dynamodb batch-write-item \
    --request-items file://batchmusic.json
```

## Create a Global Secondary Index Based on Album Title

```bash
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

## Create a Global Secondary Index Based on Artist Name and Album Title

```bash
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=Artist,AttributeType=S \
        AttributeName=AlbumTitle,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ArtistAlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"Artist\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

## Create a Global Secondary Index Based on Song Title and Year

```bash
aws dynamodb update-table \
    --table-name Music \
    --attribute-definitions\
        AttributeName=SongTitle,AttributeType=S \
        AttributeName=Year,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"SongTitleYear-index\",\"KeySchema\":[{\"AttributeName\":\"SongTitle\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Year\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

## Query Items by Artist

```bash
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '":artist":{"S":"Jorge Aragão"}}'
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '":artist":{"S":"Zeca Pagodinho"}}'
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist" \
    --expression-attribute-values  '":artist":{"S":"Dudu Nobre"}}'
```

## Query Items by Artist and Song Title

```bash
aws dynamodb query \
    --table-name Music \
    --key-condition-expression "Artist = :artist and SongTitle = :title" \
    --expression-attribute-values file://keyconditions.json
```

## Query using the Global Secondary Index Based on Album Title

```bash
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Ao Vivo 1999"}}'
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Dudu Nobre Ao Vivo"}}'
aws dynamodb query \
    --table-name Music \
    --index-name AlbumTitle-index \
    --key-condition-expression "AlbumTitle = :name" \
    --expression-attribute-values  '{":name":{"S":"Acústico MTV: Zeca Pagodinho"}}'
```

## Query using the Global Secondary Index Based on Artist Name and Album Title

```bash
aws dynamodb query \
    --table-name Music \
    --index-name ArtistAlbumTitle-index \
    --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
    --expression-attribute-values  '{":v_artist":{"S":"Jorge Aragão"},":v_title":{"S":"Ao Vivo 1999"} }'
aws dynamodb query \
    --table-name Music \
    --index-name ArtistAlbumTitle-index \
    --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
    --expression-attribute-values  '{":v_artist":{"S":"Zeca Pagodinho"},":v_title":{"S":"Acústico MTV: Zeca Pagodinho"} }'
aws dynamodb query \
    --table-name Music \
    --index-name ArtistAlbumTitle-index \
    --key-condition-expression "Artist = :v_artist and AlbumTitle = :v_title" \
    --expression-attribute-values  '{":v_artist":{"S":"Dudu Nobre"},":v_title":{"S":"Dudu Nobre Ao Vivo"} }'
```

## Query using the Global Secondary Index Based on Song Title and Year

```bash
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_title and Year = :v_year" \
    --expression-attribute-values  '{":v_title":{"S":"Ponta de dor"},":v_year":{"S":"1999"} }'
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_title and Year = :v_year" \
    --expression-attribute-values  '{":v_title":{"S":"Maneco Telecoteco"},":v_year":{"S":"2003"} }'
aws dynamodb query \
    --table-name Music \
    --index-name SongTitleYear-index \
    --key-condition-expression "SongTitle = :v_title and Year = :v_year" \
    --expression-attribute-values  '{":v_title":{"S":"Água da Minha Sede"},":v_year":{"S":"2004"} }'
```

This guide covers the creation of a DynamoDB database, inserting items, creating global secondary indexes, and querying data using various conditions. Make

 sure to replace the placeholders with your specific values.
