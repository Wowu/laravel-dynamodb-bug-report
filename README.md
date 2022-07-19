Steps to reproduce:

1. `composer install`
2. `cp .env.example .env`
3. Create sqlite database

    ```
    touch database/db.sqlite
    php artisan migrate
    ```

4. Start local dynamodb 
   
    ```
    docker run --rm -p 8000:8000 amazon/dynamodb-local:latest -jar DynamoDBLocal.jar -inMemory -sharedDb
    ````

5. Create `cache` table in dynamodb

    ```
    AWS_ACCESS_KEY_ID=test AWS_SECRET_ACCESS_KEY=test AWS_REGION=test aws dynamodb --endpoint=http://localhost:8000/ create-table --table-name cache --attribute-definitions AttributeName=key,AttributeType=S --key-schema AttributeName=key,KeyType=HASH --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
    ````

6. Start at least two worker processes

    ```
    php artisan queue:work & php artisan queue:work
    ```

7. dispatch sample job two times

    ```
    php artisan tinker --execute="\App\Jobs\SlowJob::dispatch();" && sleep 2 && php artisan tinker --execute="\App\Jobs\SlowJob::dispatch();"
    ```

8. check logs
   
   ```
   tail storage/logs/laravel.log
   #=>  [2022-07-19 12:50:18] local.INFO: Start: 2022-07-19 12:50:18  
        [2022-07-19 12:50:22] local.INFO: Start: 2022-07-19 12:50:22  
        [2022-07-19 12:50:23] local.INFO: End: 2022-07-19 12:50:23  
        [2022-07-19 12:50:27] local.INFO: End: 2022-07-19 12:50:27
   ```

   Two jobs are executed in parallel.
