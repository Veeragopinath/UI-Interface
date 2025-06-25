import React, { useState, useMemo, useRef, useCallback } from 'react';
import { AgGridReact } from 'ag-grid-react';
import Select from 'react-select';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-alpine.css';

const App = () => {
  const gridRef = useRef();
  
  const [columnDefs] = useState([
    { headerName: 'Athlete', field: 'athlete' },
    { headerName: 'Age', field: 'age' },
    { headerName: 'Country', field: 'country' },
    { headerName: 'Gold', field: 'gold' },
  ]);

  const rowData = useMemo(() => [
    { athlete: 'Michael Phelps', age: 23, country: 'USA', gold: 8 },
    { athlete: 'Natalie Coughlin', age: 25, country: 'USA', gold: 1 },
    { athlete: 'Aleksey Nemov', age: 24, country: 'Russia', gold: 2 },
    { athlete: 'Alicia Coutts', age: 26, country: 'Australia', gold: 1 },
  ], []);

  // Column options for search
  const columnOptions = [
    { label: 'Athlete', value: 'athlete' },
    { label: 'Age', value: 'age' },
    { label: 'Country', value: 'country' },
    { label: 'Gold', value: 'gold' },
  ];

  const [selectedColumn, setSelectedColumn] = useState(null);
  const [filterText, setFilterText] = useState('');

  const handleFilter = () => {
    if (selectedColumn && filterText) {
      gridRef.current.api.setQuickFilter('');
      gridRef.current.api.setFilterModel({
        [selectedColumn.value]: {
          type: 'contains',
          filter: filterText
        }
      });
    }
  };

  const clearFilter = () => {
    gridRef.current.api.setFilterModel(null);
    setFilterText('');
    setSelectedColumn(null);
  };

  return (
    <div className="ag-theme-alpine" style={{ height: 500, width: '100%' }}>
      <div style={{ marginBottom: 10, display: 'flex', gap: 10 }}>
        <Select
          options={columnOptions}
          value={selectedColumn}
          onChange={setSelectedColumn}
          placeholder="Select a column"
          isSearchable
        />
        <input
          type="text"
          placeholder="Enter filter value"
          value={filterText}
          onChange={(e) => setFilterText(e.target.value)}
        />
        <button onClick={handleFilter}>Apply Filter</button>
        <button onClick={clearFilter}>Clear</button>
      </div>

      <AgGridReact
        ref={gridRef}
        rowData={rowData}
        columnDefs={columnDefs}
        animateRows={true}
      />
    </div>
  );
};

export default App;