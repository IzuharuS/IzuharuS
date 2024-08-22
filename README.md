### Hi there 👋

import com.mongodb.MongoClient;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.AggregateIterable;
import com.mongodb.client.model.*;
import org.bson.Document;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.Date;

public class MongoBulkWriteByPeriod {
    public static void main(String[] args) {
        // Conexión a MongoDB
        MongoClient mongoClient = new MongoClient("localhost", 27017);
        MongoDatabase database = mongoClient.getDatabase("tu_base_de_datos");
        MongoCollection<Document> collection = database.getCollection("tu_coleccion");

        // Aquí comienza el pipeline de agregación
        List<Document> pipeline = new ArrayList<>();
        pipeline.add(new Document("$group", new Document("_id", "$_id")
                .append("fecha", new Document("$first", "$fecha_operacion"))  // Extraemos la fecha
                .append("totalEntradas", new Document("$sum", "$entradas"))));

        // Ejecutar el pipeline de agregación
        AggregateIterable<Document> result = collection.aggregate(pipeline);

        // Map para guardar listas de operaciones bulk por cada colección (por periodo)
        Map<String, List<WriteModel<Document>>> bulkOperationsByCollection = new HashMap<>();

        // Iterar sobre los resultados de la agregación
        for (Document doc : result) {
            // Extraer la fecha de la operación (ISODate)
            Date fechaOperacion = doc.getDate("fecha");

            // Convertir la fecha en un LocalDateTime para extraer año y mes
            LocalDateTime localDate = fechaOperacion.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
            int year = localDate.getYear();
            int month = localDate.getMonthValue();

            // Crear el nombre de la colección basada en el periodo (año y mes en formato YYYYMM)
            String collectionName = String.format("%d%02d", year, month);  // Genera el nombre, por ejemplo "202406"

            // Filtrar por el _id (para la operación de reemplazo o actualización)
            Document filter = new Document("_id", doc.get("_id"));

            // Crear operación ReplaceOneModel con upsert para actualizar o insertar el documento
            ReplaceOneModel<Document> replaceOne = new ReplaceOneModel<>(
                    filter,
                    doc,
                    new ReplaceOptions().upsert(true)
            );

            // Si la colección (por periodo) no está en el mapa, inicializar la lista de operaciones para ella
            bulkOperationsByCollection.putIfAbsent(collectionName, new ArrayList<>());

            // Añadir la operación a la lista de la colección correspondiente
            bulkOperationsByCollection.get(collectionName).add(replaceOne);
        }

        // Iterar sobre las colecciones y ejecutar las operaciones bulkWrite por cada una
        for (Map.Entry<String, List<WriteModel<Document>>> entry : bulkOperationsByCollection.entrySet()) {
            String collectionName = entry.getKey();
            List<WriteModel<Document>> bulkOperations = entry.getValue();

            // Obtener la colección por nombre
            MongoCollection<Document> targetCollection = database.getCollection(collectionName);

            // Ejecutar el bulkWrite en la colección correspondiente
            if (!bulkOperations.isEmpty()) {
                BulkWriteResult bulkWriteResult = targetCollection.bulkWrite(bulkOperations);

                // Mostrar los resultados del bulkWrite
                System.out.println("Colección: " + collectionName);
                System.out.println("Inserted: " + bulkWriteResult.getInsertedCount());
                System.out.println("Modified: " + bulkWriteResult.getModifiedCount());
                System.out.println("Matched: " + bulkWriteResult.getMatchedCount());
                System.out.println("Upserts: " + bulkWriteResult.getUpserts());
            }
        }

        // Cerrar la conexión al cliente
        mongoClient.close();
    }
}
