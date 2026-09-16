# Data model integration in Editor

If target code repo contains definitions of data entities, we can incorporate field names into the feature file.

Ex: @FirstName, @EmployeeNumber

Support 'standard' Java POJOs and SAP CDS data layout.

## Choose entity field @v1 [proposed]

When we enter '@' and press tab, the app presents a searchable list of entity fields.

If two entities share the same field name, it will be prefixed by entity name. Ex: '@User.FirstName'

[test: reads the instance fields of a plain POJO, whatever their modifiers : https://github.com/SpecDriven/specdriven_app/blob/main/tests/entities.test.ts#L27 ]
[test: reads an entity's element names, associations and compositions included : https://github.com/SpecDriven/specdriven_app/blob/main/tests/entities.test.ts#L127 ]
[test: offers a unique field bare, and a shared one prefixed by each entity : https://github.com/SpecDriven/specdriven_app/blob/main/tests/entities.test.ts#L208 ]
[test: lists the code repo's fields, shared names prefixed by their entity : https://github.com/SpecDriven/specdriven_app/blob/main/tests/entities.test.ts#L308 ]
[test: finds the @ run the caret ends, with what was typed after the @ : https://github.com/SpecDriven/specdriven_app/blob/main/tests/entityFields.test.tsx#L29 ]
[test: matches case-insensitively, prefixes before mere mentions : https://github.com/SpecDriven/specdriven_app/blob/main/tests/entityFields.test.tsx#L55 ]
[test: replaces the @ run with the choice and puts the caret after it : https://github.com/SpecDriven/specdriven_app/blob/main/tests/entityFields.test.tsx#L73 ]
