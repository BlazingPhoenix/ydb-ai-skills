# FloatVector parameters in Python (`ydb`)

For an application-provided `list[float]`, encode little-endian Float32 values and append the `0x01` FloatVector marker before binding:

```python
import struct
import ydb

embedding_bytes = struct.pack(f"<{len(embedding)}f", *embedding) + b"\x01"
result = pool.execute_with_retries(
    "DECLARE $embedding AS String; SELECT Knn::CosineDistance(embedding, $embedding) FROM Facts;",
    {"$embedding": (embedding_bytes, ydb.PrimitiveType.String)},
)
```

Use `String` for a stored embedding column or an `AS_TABLE` batch member too. Passing `List<Float>` and calling `Knn::ToBinaryStringFloat` in YQL is the slower alternative when the vector already exists in Python. The YQL conversion remains useful for vectors constructed inside YQL.

Sources: <https://ydb.tech/docs/en/recipes/ydb-sdk/vector-search?version=main> (Python recommended approach); <https://ydb.tech/docs/en/yql/reference/udf/list/knn#functions-convert>.
