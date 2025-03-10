---
title: Sorting
id: sorting
---

## State

Sorting state is stored on the table instance using the following shape:

```tsx
export type SortDirection = 'asc' | 'desc'

export type ColumnSort = {
  id: string
  desc: boolean
}

export type SortingState = ColumnSort[]

export type SortingTableState = {
  sorting: SortingState
}
```

## Sorting Functions

The following sorting functions are built-in to the table core:

- `alphanumeric`
  - Sorts by mixed alphanumeric values without case-sensitivity. Slower, but more accurate if your strings contain numbers that need to be naturally sorted.
- `alphanumericCaseSensitive`
  - Sorts by mixed alphanumeric values with case-sensitivity. Slower, but more accurate if your strings contain numbers that need to be naturally sorted.
- `text`
  - Sorts by text/string values without case-sensitivity. Faster, but less accurate if your strings contain numbers that need to be naturally sorted.
- `textCaseSensitive`
  - Sorts by text/string values with case-sensitivity. Faster, but less accurate if your strings contain numbers that need to be naturally sorted.
- `datetime`
  - Sorts by time, use this if your values are `Date` objects.
- `basic`
  - Sorts using a basic/standard `a > b ? -1 : b < a ? 1 : 0` comparison. This is the fastest sorting function, but may not be the most accurate.

Every sorting function receives 2 rows and a column ID and are expected to compare the two rows using the column ID to return `-1`, `0`, or `1` in ascending order. Here's a cheat sheet:

| Return | Ascending Order |
| ------ | --------------- |
| `-1`   | `a < b`         |
| `0`    | `a === b`       |
| `1`    | `a > b`         |

This is the type signature for every sorting function:

```tsx
export type SortingFn<TGenerics extends TableGenerics> = {
  (rowA: Row<TGenerics>, rowB: Row<TGenerics>, columnId: string): number
}
```

#### Using Sorting Functions

Sorting functions can be used/referenced/defined by passing the following to `columnDefinition.sortingFn`:

- A `string` that references a built-in sorting function
- A `string` that references a custom sorting functions provided via the `tableOptions.sortingFns` option
- A function directly provided to the `columnDefinition.sortingFn` option

The final list of sorting functions available for the `columnDef.sortingFn` use the following type:

```tsx
export type SortingFnOption<TGenerics extends TableGenerics> =
  | 'auto'
  | BuiltInSortingFn
  | keyof TGenerics['SortingFns']
  | SortingFn<TGenerics>
```

## Column Def Options

### `sortingFn`

```tsx
sortingFn?: SortingFn | keyof TGenerics['SortingFns'] | keyof BuiltInSortingFns
```

The sorting function to use with this column.

Options:

- A `string` referencing a [built-in sorting function](#sorting-functions))
- A `string` referencing a custom sorting function defined on the `sortingFns` table option
- A [custom sorting function](#sorting-functions)

### `sortDescFirst`

```tsx
sortDescFirst?: boolean
```

Set to `true` for sorting toggles on this column to start in the descending direction.

### `enableSorting`

```tsx
enableSorting?: boolean
```

Enables/Disables sorting for this column.

### `enableMultiSort`

```tsx
enableMultiSort?: boolean
```

Enables/Disables multi-sorting for this column.

### `invertSorting`

```tsx
invertSorting?: boolean
```

Inverts the order of the sorting for this column. This is useful for values that have an inverted best/worst scale where lower numbers are better, eg. a ranking (1st, 2nd, 3rd) or golf-like scoring

### `sortUndefined`

```tsx
sortUndefined?: false | -1 | 1 // defaults to false
```

| Value   | Description                                                                                                                    |
| ------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `false` | Undefined values will be considered tied and need to be sorted by the next colum filter or original index (which ever applies) |
| `-1`    | Undefined values will be sorted with higher priority (ascending)                                                               |
| `1`     | Undefined values will be sorted with lower priority (descending)                                                               |

## Column API

### `getAutoSortingFn`

```tsx
getAutoSortingFn: () => SortingFn<TGenerics>
```

Returns a sorting function automatically inferred based on the columns values.

### `getAutoSortDir`

```tsx
getAutoSortDir: () => SortDirection
```

Returns a sort direction automatically inferred based on the columns values.

### `getSortingFn`

```tsx
getSortingFn: () => SortingFn<TGenerics>
```

Returns the resolved sorting function to be used for this column

### `getCanSort`

```tsx
getCanSort: () => boolean
```

Returns whether this column can be sorted.

### `getCanMultiSort`

```tsx
getCanMultiSort: () => boolean
```

Returns whether this column can be multi-sorted.

### `getSortIndex`

```tsx
getSortIndex: () => number
```

Returns the index position of this column's sorting within the sorting state

### `getIsSorted`

```tsx
getIsSorted: () => false | SortDirection
```

Returns whether this column is sorted.

### `clearSorting`

```tsx
clearSorting: () => void
```

Removes this column from the table's sorting state

### `toggleSorting`

```tsx
toggleSorting: (desc?: boolean, isMulti?: boolean) => void
```

Toggles this columns sorting state. If `desc` is provided, it will force the sort direction to that value. If `isMulti` is provided, it will additivity multi-sort the column (or toggle it if it is already sorted).

### `getToggleSortingHandler`

```tsx
getToggleSortingHandler: () => undefined | ((event: unknown) => void)
```

Returns a function that can be used to toggle this column's sorting state. This is useful for attaching a click handler to the column header.

## Table Options

### `manualSorting`

```tsx
manualSorting?: boolean
```

Enables manual sorting for the table. If this is `true`, you will be expected to sort your data before it is passed to the table instance. This is useful if you are doing server-side sorting.

### `sortingFns`

```tsx
sortingFns?: TGenerics['SortingFns']
```

Normally set ahead of time when using the `createTable()` helper, this option allows you to define custom sorting functions that can be referenced by their string key.

Example:

```tsx
const table = createTable().setOptions({
  sortingFns: {
    myCustomSortingFn: (rows, columnIds, sortingValue) => {
      ///
    },
  },
})

const column = table.createDataColumn('key', {
  sortingFn: 'myCustomSortingFn',
})
```

### `onSortingChange`

```tsx
onSortingChange?: OnChangeFn<SortingState>
```

If provided, this function will be called with an `updaterFn` when `state.sorting` changes. This overrides the default internal state management, so you will need to persist the state change either fully or partially outside of the table.

### `enableSorting`

```tsx
enableSorting?: boolean
```

Enables/Disables sorting for the table.

### `enableSortingRemoval`

```tsx
enableSortingRemoval?: boolean
```

Enables/Disables the ability to remove sorting for the table.

### `enableMultiRemove`

```tsx
enableMultiRemove?: boolean
```

Enables/disables the ability to remove multi-sorts

### `enableMultiSort`

```tsx
enableMultiSort?: boolean
```

Enables/Disables multi-sorting for the table.

### `sortDescFirst`

```tsx
sortDescFirst?: boolean
```

If `true`, all sorts will default to descending as their first toggle state.

### `getSortedRowModel`

```tsx
getSortedRowModel?: (instance: TableInstance<TGenerics>) => () => RowModel<TGenerics>
```

This function is used to retrieve the sorted row model. If using server-side sorting, this function is not required. To use client-side sorting, pass the exported `getSortedRowModel()` from your adapter to your table instance or implement your own.

### `maxMultiSortColCount`

```tsx
maxMultiSortColCount?: number
```

Set a maximum number of columns that can be multi-sorted.

### `isMultiSortEvent`

```tsx
isMultiSortEvent?: (e: unknown) => boolean
```

Pass a custom function that will be used to determine if a multi-sort event should be triggered. It is passed the event from the sort toggle handler and should return `true` if the event should trigger a multi-sort.

## Table Instance API

### `setSorting`

```tsx
setSorting: (updater: Updater<SortingState>) => void
```

Sets or updates the `state.sorting` state.

### `resetSorting`

```tsx
resetSorting: (defaultState?: boolean) => void
```

Resets the **sorting** state to `initialState.sorting`, or `true` can be passed to force a default blank state reset to `[]`.

### `getPreSortedRowModel`

```tsx
getPreSortedRowModel: () => RowModel<TGenerics>
```

Returns the row model for the table before any sorting has been applied.

### `getSortedRowModel`

```tsx
getSortedRowModel: () => RowModel<TGenerics>
```

Returns the row model for the table after sorting has been applied.
