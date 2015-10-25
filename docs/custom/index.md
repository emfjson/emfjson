---
layout: doc
title: Customizing Features
---

## Custom Id Serializer

```java
public class CDOIDSerializer implements IdSerializer {

	@Override
	public void serialize(EObject object, JsonGenerator jg, SerializerProvider provider) 
	throws IOException {
	
	    final JacksonOptions options = (JacksonOptions) provider.getAttribute("options");
	    final CDOObject cdoObject = CDOUtil.getCDOObject(object);
	    final CDOID cdoid = CDOIDUtil.getCDOID(cdoObject);

		jg.writeNumberField(options.idField, CDOIDUtil.getLong(cdoid));
	}

}
```

## Custom Reference Serializer

```java
public class CDOReferenceSerializer implements ReferenceSerializer {

	@Override
	public void serialize(EObject source, EObject value, JsonGenerator jg, SerializerProvider provider) 
	throws IOException {

        final CDOObject cdoObject = CDOUtil.getCDOObject(value);
        final CDOID cdoid = CDOIDUtil.getCDOID(cdoObject);

        if (cdoid != null) {
            jg.writeNumber(CDOIDUtil.getLong(cdoid));
        } else {
            jg.writeNull();
        }
	}

}
```

## Custom Reference DeSerializer

```java
public class CDOReferenceDeserializer implements ReferenceDeserializer {

	private final CDOTransaction transaction;

	public CDOReferenceDeserializer(CDOTransaction transaction) {
		this.transaction = transaction;
	}

	@Override
	public ReferenceEntry deserialize(JsonParser jp, final EObject owner, final EReference reference, DeserializationContext ctxt) throws IOException {
		if (JsonToken.VALUE_NUMBER_INT.equals(jp.getCurrentToken())) {
			final JsonLocation location = jp.getCurrentLocation();
			final long value = jp.getLongValue();

			return new ReferenceEntry() {
				@Override
				public void resolve(ResourceSet resourceSet, URIHandler handler, ReferenceEntries re) {
					CDOID cdoid = CDOIDUtil.createLong(value);

					EObject object;
					try {
						object = transaction.getObject(cdoid);
					} catch (Exception e) {
						owner.eResource().getErrors().add(new JSONException(e, location));
						return;
					}

					if (object instanceof CDOLegacyAdapter) {
						object = ((CDOLegacyAdapter) object).cdoInternalInstance();
					}

					try {
						EObjects.setOrAdd(owner, reference, object);
					} catch (Exception e) {
						owner.eResource().getErrors().add(new JSONException(e, location));
					}
				}
			};
		}

		return null;
	}

}
```

## Custom Type Serializer

## Custom Type DeSerializer

