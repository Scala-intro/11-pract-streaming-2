# 11 Practica Streaming + SQL

1. [Arrancar Zeppelin ](#schema1)
2. [Importación de librerías ](#schema2)
3. [Creando las variables de conexión](#schema3)
4. [Crear socket](#schema4)
5. [Creamos la clase Registro](#schema5)
6. [Creamos el contador de palabras](#schema6)
7. [Ponemos a la escucha el programa](#schema7)

6. [Instalamos netcat](#schema6)
7. [Terminar el proceso](#schema7)


<hr>

<a name="schema1"></a>

# 1. Arrancar Zeppelin
Navegamos en la consola hasta llegar donde tenemos descargados la carpeta Zeppelin y ejecutamos:
~~~
bin/zeppelin-daemon.sh start
~~~

Seguidamente abrimos un página en el navegador y vamos a `http://localhost:8080`, se nos abre zeppelin y creamos un nuevo notebook, llamado Temperatura Sensor y como intérprete elegimos `spark2`
<hr>

<a name="schema2"></a>

# 2. Importación de librerías

~~~scala
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.SparkSession
import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming.{Seconds, StreamingContext,Time}
~~~
<hr>

<a name="schema3"></a>

# 3. Creando las variables de conexión

`sparkConf` crea la configuración y le da el nombre a la aplicación
`scc` = streaming spakr context, variable que cada 1 segundo va a recoger los datos.

~~~scala
val ssc = new StreamingContext(sc, Seconds(2))
~~~
<hr>

<a name="schema4"></a>


# 4. Crear socket
La notación  `._split(" ")` es lo mismo que `linea => linea.split(" ")`
~~~scala
val lineas = ssc.socketTextStream("localhost", 9000, StorageLevel.MEMORY_AND_DISK_SER)
val palabras = lineas.flatMap(_.split(" "))

~~~
<hr>

<a name="schema5"></a>

# 5. Creamos la clase Registro

Es una clase que recibe un palabra
~~~scala
case class Registro(palabra:String)
~~~



<hr>

<a name="schema6"></a>

# 6. Creamos el contador de palabras
~~~scala
palabras.foreachRDD{(rdd:RDD[String], time:Time) => 
    import spark.implicits._
    val palabrasDF = rdd.map(p => Registro(p)).toDF()
    palabrasDF.createOrReplaceTempView("palabras") //vista temporal
    val contadorPalabraDF = spark.sql("select palabra, count(*) as total from palabras group by palabra")
    println(s"*****${time}******************")
    contadorPalabraDF.show()
}
~~~



<hr>

<a name="schema7"></a>

# 7. Ponemos a la escucha el programa
~~~scala
ssc.start()
ssc.awaitTermination()
~~~