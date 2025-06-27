Below is a complete example of a single-file React component using MUI X DataGridPremium with a custom column filter that includes a searchable dropdown (via <Autocomplete>). Paste this into your .tsx or .jsx file and adjust imports/versions as needed:

import * as React from 'react';
import { DataGridPremium, GridColDef, GridFilterOperator, useGridApiContext } from '@mui/x-data-grid-premium';
import { Autocomplete, InputBase, Box } from '@mui/material';
import { getGridSingleSelectOperators } from '@mui/x-data-grid';

interface Row {
  id: number;
  category: string;
  value: string;
}

const rows: Row[] = [
  { id: 1, category: 'Fruit', value: 'Apple' },
  { id: 2, category: 'Fruit', value: 'Banana' },
  { id: 3, category: 'Vegetable', value: 'Carrot' },
  { id: 4, category: 'Vegetable', value: 'Broccoli' },
];

const categories = ['Fruit', 'Vegetable', 'Grain', 'Dairy'];

// Custom InputComponent using Autocomplete
function CustomAutocompleteInput(props: any) {
  const { item, applyValue, focusElementRef, apiRef } = props;
  const column = apiRef.current.getColumn(item.columnField!);
  const options: string[] = (column.valueOptions as string[]) ?? [];

  const handleChange = (_: any, newVal: string | null) => {
    applyValue({ ...item, value: newVal });
  };

  return (
    <Box sx={{ px: 1, py: 0.5 }}>
      <Autocomplete
        disableClearable
        fullWidth
        options={options}
        value={item.value ?? ''}
        onChange={handleChange}
        renderInput={(params) => (
          <InputBase
            {...params}
            inputRef={focusElementRef}
            placeholder="Search…"
            sx={{ width: '100%' }}
          />
        )}
      />
    </Box>
  );
}

// Custom filter operator definition
const searchableSingleSelectOperator: GridFilterOperator = {
  label: 'is',
  value: 'is',
  getApplyFilterFn: (filterItem) => {
    if (!filterItem.columnField || filterItem.value == null) return null;
    return ({ value }) => value === filterItem.value;
  },
  InputComponent: CustomAutocompleteInput,
};

export default function App() {
  const columns: GridColDef[] = [
    {
      field: 'category',
      headerName: 'Category',
      width: 180,
      type: 'singleSelect',
      valueOptions: categories,
      filterOperators: [searchableSingleSelectOperator],
    },
    {
      field: 'value',
      headerName: 'Value',
      width: 180,
    },
  ];

  return (
    <div style={{ height: 400, width: '100%' }}>
      <DataGridPremium
        rows={rows}
        columns={columns}
        filterMode="client"
        disableColumnSelector
      />
    </div>
  );
}

🔍 Explanation

1. CustomAutocompleteInput
Renders an <Autocomplete> inside the filter dropdown, connected to the grid via apiRef, item, and applyValue.


2. searchableSingleSelectOperator
Defines a filter operator with label "is" that compares equality and uses our custom InputComponent.


3. Column Setup
The category column is configured as singleSelect, uses the custom operator, and provides valueOptions.


4. Usage
The grid (DataGridPremium) is set to use client-side filtering, showing our searchable dropdown in the filter menu.



This approach is based on the MUI documentation for customizing filter operators and inputs  . Let me know if you’d like to extend this to multi-select filters or server-side filtering!

