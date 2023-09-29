# Exercise 2
In this exercise, we will create a FilterBarDelegate, add a FilterBar to the xml view, use the filter association of the table and implement the search feature as a combination of filters.
## Step 1: Create an FilterBarDelegate
This delegate is responsible for managing the filtering logic for an application. This includes creating and managing filter fields, as well as providing the correpsonding PropertyInfo for filter fields.
###### delegate/JSONFilterBarDelegate.js
```javascript
sap.ui.define([
	"sap/ui/mdc/FilterBarDelegate",
	"mdc/tutorial/model/metadata/JSONPropertyInfo",
	"sap/ui/mdc/FilterField",
	"sap/ui/core/Core"
	"sap/ui/core/Fragment"
], function (FilterBarDelegate, JSONPropertyInfo, FilterField, Core, Fragment) {
	"use strict";

	const JSONFilterBarDelegate = Object.assign({}, FilterBarDelegate);

	JSONFilterBarDelegate.fetchProperties = function () {
		return Promise.resolve(JSONPropertyInfo);
	};

	JSONFilterBarDelegate.addItem = function(oFilterBar, sPropertyName) {
		const oProperty = JSONPropertyInfo.find((oPropertyInfo) => oPropertyInfo.name === sPropertyName);
		return _addFilterField(oProperty, oFilterBar);
	};

	JSONFilterBarDelegate.removeItem = function(oFilterBar, oFilterField) {
		oFilterField.destroy();
		return Promise.resolve(true);
	};

	function _addFilterField(oProperty, oFilterBar) {
		const sName = oProperty.name;
		const sFilterFieldId = oFilterBar.getId() + "--filter--" + sName;
		let oFilterField = Core.byId(sFilterFieldId);
		let pFilterField;

		if (oFilterField) {
			pFilterField = Promise.resolve(oFilterField);
		} else {
			oFilterField = new FilterField(sFilterFieldId, {
				dataType: oProperty.dataType,
				conditions: "{$filters>/conditions/" + sName + '}',
				propertyKey: sName,
				required: oProperty.required,
				label: oProperty.label,
				maxConditions: oProperty.maxConditions,
				delegate: { name: "sap/ui/mdc/field/FieldBaseDelegate", payload: {} }
			});
			pFilterField = Promise.resolve(oFilterField);
		}
		return pFilterField;
	}

	return JSONFilterBarDelegate;
}, /* bExport= */false);
```

## Step 2: Use the MDC Filter Bar
To add a FilterBar to the XML view, we can use the [`sap.ui.mdc.FilterBar`](https://sdk.openui5.org/api/sap.ui.mdc.FilterBar) control. Setting the previously created delegate makes sure, that the FilterBar can deal with the specific JSON data we are facing. Place the FilterBar inside of the DynamicPageHeader.
###### view/Mountains.view.xml
```xml
				<mdc:FilterBar id="filterbar" delegate="{name: 'mdc/tutorial/delegate/JSONFilterBarDelegate'}"
						p13nMode = "Item,Value">
					<mdc:basicSearchField>
						<mdc:FilterField delegate="{name: 'sap/ui/mdc/field/FieldBaseDelegate'}"
							dataType="sap.ui.model.type.String"
							placeholder= "Search Mountains"
							conditions="{$filters>/conditions/$search}"
							maxConditions="1"/>
					</mdc:basicSearchField>
					<mdc:filterItems>
						<mdc:FilterField
							label="Name"
							propertyKey="name"
							dataType="sap.ui.model.type.String"
							conditions="{$filters>/conditions/name}"
							delegate="{name: 'sap/ui/mdc/field/FieldBaseDelegate'}"/>
					</mdc:filterItems>
					<mdc:dependents>
					
					</mdc:dependents>
				</mdc:FilterBar>
```

Use the filter association of the table to connect it to the filter bar.
###### view/Mountains.view.xml
```xml
			<mdc:Table
				id="table"
				header="Mountains"
				p13nMode="Sort,Column"
				type="ResponsiveTable"
				threshold="100"
				filter="filterbar"
				showRowCount="false"
				delegate="{
					name: 'mdc/tutorial/delegate/JSONTableDelegate',
					payload: {
						bindingPath: 'mountains>/mountains'
					}
				}">
```

## Step 3: Enable the Search in the JSONTableDelegate
To implement the search feature, we need to extend the `JSONTableDelegate` and override the `getFilters` method. We implement a simple search feature by combining several filters and appending them to the regular filter set, which is prepared by the `TableDelegate`.
###### delegate/JSONTableDelegate.js
```javascript
	JSONTableDelegate.getFilters = function(oTable) {
		const aSearchFilters = _createSearchFilters(Core.byId(oTable.getFilter()).getSearch());
		return TableDelegate.getFilters(oTable).concat(aSearchFilters);
	};

	function _createSearchFilters(sSearch) {
		let aFilters = [];
		if (sSearch) {
			const aPaths = ["name", "range", "parent_mountain", "countries"];
			aFilters = aPaths.map(function (sPath) {
				return new Filter({
					path: sPath,
					operator: FilterOperator.Contains,
					value1: sSearch
				});
			});
			aFilters = [new Filter(aFilters, false)];
		}
		return aFilters;
	}
```
Go and try out the filter and search functionality in your application. The table should display only the filtered items! 🙌

![Exercise 2 Result](ex2.png)

## Summary
In this exercise, we have extended the functionality of our application by adding a FilterBarDelegate to handle filtering operations, and a JSONTableDelegate to handle search operations. We have also learned how to use the filter association of the table to connect the FilterBar to the Table. This allows us to create a more interactive and dynamic user interface, where the user can filter and search the data in the table based on their needs.

Continue to - [Exercise 3](../ex3/readme.md)