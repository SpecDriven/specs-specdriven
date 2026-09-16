# Data model integration in Editor

If target code repo contains definitions of data entities, we can incorporate field names into the feature file.

Ex: @FirstName, @EmployeeNumber

Support 'standard' Java POJOs and SAP CDS data layout.

## Choose entity field @v1 [proposed]

When we enter '@' and press tab, the app presents a searchable list of entity fields.

If two entities share the same field name, it will be prefixed by entity name. Ex: '@User.FirstName'
