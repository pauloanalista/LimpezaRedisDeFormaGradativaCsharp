# LimpezaRedisDeFormaGradativaCsharp
Limpeza Redis de forma gradativa em C#



```csharp
public async Task ExpirarTodos()
 {
      string conexao = _configuration.GetConnectionString("redis");
      string[] vetConexao = conexao.Split(",");
      using (ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(conexao))
      {
          long secondsToExpire = 10;
          long dayInSeconds = 86400;
          //Parallel.ForEach(redis.GetEndPoints(), async ep =>
          //{
              int nextCursor = 0;
              do
              {
                  var server = redis.GetServer(vetConexao[0]);
                  var redisResult = server.Execute("SCAN", new object[] { nextCursor.ToString(), "MATCH", "*", "COUNT", "100" });
                  var innerResult = (RedisResult[])redisResult;
                  nextCursor = int.Parse((string)innerResult[0]);
                  IEnumerable<string> resultLines = ((string[])innerResult[1]);
                  foreach (var key in resultLines)
                  {
                      //Obtem dados e seta expiração
                      string value = RedisCache.GetString(key);

                      //Setar tempo de expiração
                      var options = new DistributedCacheEntryOptions();
                      options.SetAbsoluteExpiration(TimeSpan.FromSeconds(secondsToExpire));

                      //Atualiza chave Redis
                      await RedisCache.SetStringAsync(key, value, options);

                      //Imprime no console
                      Debug.WriteLine($@"secondsToExpire:{secondsToExpire} Chave: " + key);

                      secondsToExpire += 10;
                  }
                  //Se for 24 horas

                  if (secondsToExpire > dayInSeconds)
                  {
                      //Reseta contador
                      secondsToExpire = 10;
                  }
              }
              while (nextCursor != 0);
         // });
      }
  }
  ```
