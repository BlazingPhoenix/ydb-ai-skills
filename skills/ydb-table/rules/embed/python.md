# Python SDK (`ydb`) — vector parameter anti-patterns

### RULE-PY-01: Client FloatVector passed as `List<Float>` for server conversion

**Severity**: Medium

**What to look for**: an application-provided `list[float]` is bound as `ydb.ListType(ydb.PrimitiveType.Float)`, while the YQL query calls `Knn::ToBinaryStringFloat($embedding)` or converts an `AS_TABLE($items)` embedding member.

**Problem**: each coordinate is serialized as a list element, transmitted, and converted to the binary FloatVector format by YDB on every call.

**Fix**: encode the vector with `struct.pack(f"<{len(vector)}f", *vector) + b"\x01"`, bind the resulting `bytes` as `ydb.PrimitiveType.String`, and declare the YQL parameter or batch member as `String`. Use the parameter directly for storage or `Knn` distance functions. Keep the YQL converter when the vector is constructed in YQL; other vector types need their own formats.

**Source**: <https://ydb.tech/docs/en/recipes/ydb-sdk/vector-search?version=main> (Python recommended approach); <https://ydb.tech/docs/en/yql/reference/udf/list/knn#functions-convert>.
