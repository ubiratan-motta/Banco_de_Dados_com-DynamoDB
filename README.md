# Criação de Banco de Dados com DynamoDB e AWS CLI
- Criação do Banco de Dados
- Incluindo arquivos JSON no BD
- Criação de Index para Queries mais complexas
- Uso de Queries para extração de dados


## Criar uma tabela
    aws dynamodb create-table \
        --table-name Musica \
        --attribute-definitions \
            AttributeName=Artista,AttributeType=S \
            AttributeName=TituloMusica,AttributeType=S \
        --key-schema \
            AttributeName=Artista,KeyType=HASH \
            AttributeName=TituloMusica,KeyType=RANGE \
        --provisioned-throughput \
            ReadCapacityUnits=10,WriteCapacityUnits=5


## Inserir um item
    aws dynamodb put-item \
        --table-name Musica \
        --item file://itemmusic.json \


## Inserir múltiplos itens
    aws dynamodb batch-write-item \
        --request-items file://batchmusic.json


## Criar um index global secundário baeado no título do álbum
    aws dynamodb update-table \
        --table-name Musica \
        --attribute-definitions AttributeName=TituloAlbum,AttributeType=S \
        --global-secondary-index-updates \
            "[{\"Create\":{\"IndexName\": \"TituloAlbum-index\",\"KeySchema\":[{\"AttributeName\":\"TituloAlbum\",\"KeyType\":\"HASH\"}], \
            \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"


## Criar um index global secundário baseado no nome do artista e no título do álbum
    aws dynamodb update-table \
        --table-name Musica \
        --attribute-definitions\
            AttributeName=Artista,AttributeType=S \
            AttributeName=TituloAlbum,AttributeType=S \
        --global-secondary-index-updates \
            "[{\"Create\":{\"IndexName\": \"ArtistaTituloAlbum-index\",\"KeySchema\":[{\"AttributeName\":\"Artista\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"TituloAlbum\",\"KeyType\":\"RANGE\"}], \
            \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"


## Criar um index global secundário baseado no título da música e no ano
    aws dynamodb update-table \
        --table-name Musica \
        --attribute-definitions\
            AttributeName=TituloMusica,AttributeType=S \
            AttributeName=AnoMusica,AttributeType=S \
        --global-secondary-index-updates \
            "[{\"Create\":{\"IndexName\": \"TituloMusicaAno-index\",\"KeySchema\":[{\"AttributeName\":\"TituloMusica\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"AnoMusica\",\"KeyType\":\"RANGE\"}], \
            \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"


## Pesquisar item por artista
    aws dynamodb query \
        --table-name Musica \
        --key-condition-expression "Artista = :artista" \
        --expression-attribute-values  '":artista":{"S":"Jorge Aragão"}}'
    aws dynamodb query \
        --table-name Musica \
        --key-condition-expression "Artista = :artista" \
        --expression-attribute-values  '":artista":{"S":"Zeca Pagodinho"}}'
    aws dynamodb query \
        --table-name Musica \
        --key-condition-expression "Artista = :artista" \
        --expression-attribute-values  '":artista":{"S":"Dudu Nobre"}}'


##  Pesquisar item por artista e título da música
    aws dynamodb query \
        --table-name Musica \
        --key-condition-expression "Artista = :artista and TituloMusica = :titulo" \
        --expression-attribute-values file://keyconditions.json


## Pesquisa pelo index secundário baseado no título do álbum
    aws dynamodb query \
        --table-name Musica \
        --index-name TituloAlbum-index \
        --key-condition-expression "TituloAlbum = :name" \
        --expression-attribute-values  '{":name":{"S":"Ao Vivo 1999"}}'
    aws dynamodb query \
        --table-name Musica \
        --index-name TituloAlbum-index \
        --key-condition-expression "TituloAlbum = :name" \
        --expression-attribute-values  '{":name":{"S":"Dudu Nobre Ao Vivo"}}'
    aws dynamodb query \
        --table-name Musica \
        --index-name TituloAlbum-index \
        --key-condition-expression "TituloAlbum = :name" \
        --expression-attribute-values  '{":name":{"S":"Acústico MTV: Zeca Pagodinho"}}'


## Pesquisa pelo index secundário baseado no nome do artista e no título do álbum
    aws dynamodb query \
        --table-name Musica \
        --index-name ArtistaTituloAlbum-index \
        --key-condition-expression "Artista = :v_artista and TituloAlbum = :v_titullo" \
        --expression-attribute-values  '{":v_artista":{"S":"Jorge Aragão"},":v_titullo":{"S":"Ao Vivo 1999"} }'
    aws dynamodb query \
        --table-name Musica \
        --index-name ArtistaTituloAlbum-index \
        --key-condition-expression "Artista = :v_artista and TituloAlbum = :v_titullo" \
        --expression-attribute-values  '{":v_artista":{"S":"Zeca Pagodinho"},":v_titullo":{"S":"Acústico MTV: Zeca Pagodinho"} }'
    aws dynamodb query \
        --table-name Musica \
        --index-name ArtistaTituloAlbum-index \
        --key-condition-expression "Artista = :v_artista and TituloAlbum = :v_titullo" \
        --expression-attribute-values  '{":v_artista":{"S":"Dudu Nobre"},":v_titullo":{"S":"Dudu Nobre Ao Vivo"} }'


## Pesquisa pelo index secundário baseado no título da música e no ano
    aws dynamodb query \
        --table-name Musica \
        --index-name TituloMusicaAno-index \
        --key-condition-expression "TituloMusica = :v_musica and AnoMusica = :v_ano" \
        --expression-attribute-values  '{":v_musica":{"S":"Ponta de dor"},":v_ano":{"S":"1999"} }'
    aws dynamodb query \
        --table-name Musica \
        --index-name TituloMusicaAno-index \
        --key-condition-expression "TituloMusica = :v_musica and AnoMusica = :v_ano" \
        --expression-attribute-values  '{":v_musica":{"S":"Maneco Telecoteco"},":v_ano":{"S":"2003"} }'
    aws dynamodb query \
        --table-name Musica \
        --index-name TituloMusicaAno-index \
        --key-condition-expression "TituloMusica = :v_musica and AnoMusica = :v_ano" \
        --expression-attribute-values  '{":v_musica":{"S":"Agua da Minha Sede"},":v_ano":{"S":"2004"} }'
