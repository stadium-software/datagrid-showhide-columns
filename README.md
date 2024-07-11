# DataGrid Show / Hide Columns

DataGrids sometimes contain a long list of fields and can get very wide. This repo contains a module that allows users to hide columns they don't want to see. Customisations to the column display are automatically saved per visitor and reapplied when the visitor returns. 

https://github.com/stadium-software/datagrid-showhide-columns/assets/2085324/bf547320-b77f-49bd-b03b-df9fa9ca2d6f

# Version 
1.1 Added logic to detect uniqueness of DataGrid class on page

1.2 Fixed Selectable column & hidden column bugs

1.3 Added column reordering feature; JS optimisations; CSS updates; Various bug fixes

1.4 Using DataModel instead of DOM; switched initially hidden input list from column numbers to names; added EnableReordering boolean to switch reorder feature on/off; fixed minor display bugs

## Application Setup
1. Check the *Enable Style Sheet* checkbox in the application properties

## Database, Connector and DataGrid
1. Use the instructions from [this repo](https://github.com/stadium-software/samples-database) to setup the database and DataGrid for this sample

## Global Script Setup
1. Create a Global Script called "ColumnDisplay"
2. Add input parameters below to the Global Script
   1. DataGridClass
   2. InitialHiddenColumns
   3. EnableReordering
3. Drag a *JavaScript* action into the script
4. Add the Javascript below into the JavaScript code property
```javascript
/* Stadium Script Version 1.4 https://github.com/stadium-software/datagrid-showhide-columns */
let inputClass = ~.Parameters.Input.DataGridClass;
let dgClassName = "." + inputClass;
let enableReorder = ~.Parameters.Input.EnableReordering;
let colsParameter = ~.Parameters.Input.InitialHiddenColumns;
let scope = this;
let dg = document.querySelectorAll(dgClassName);
if (dg.length == 0) {
    console.error("No control with the class '" + inputClass + "' was found");
    return false;
} else if (dg.length > 1) {
    console.error("The class '" + inputClass + "' is assigned to multiple DataGrids. DataGrids using this script must have unique classnames");
    return false;
} else { 
    dg = dg[0];
}
const pathname = window.location.pathname.replace("/", "");
/*Hide columns as per cookie*/
let hideCols = [];
if (Array.isArray(colsParameter)) hideCols = colsParameter;
let cookieCols = getCookie(pathname + "-" + inputClass + "-hidden-columns");
if (cookieCols !== null && cookieCols !== "") {
    hideCols = cookieCols.split(",");
} else if (cookieCols == "") {
    hideCols = [];
}
/*Reorder columns as per cookie*/
let arrColOrder = [];
let cookieColOrder = getCookie(pathname + "-" + inputClass + "-column-order");
if (cookieColOrder !== null) {
    arrColOrder = cookieColOrder.split(",");
}
let arrColumnDefinitions = getDMValues(dg, "ColumnDefinitions");
for (let i = 0; i < arrColOrder.length; i++) {
    arrColumnDefinitions = moveItem(arrColumnDefinitions,i,getColItemIndex(arrColOrder[i]));
}
initDataGridHide();

function initDataGridHide(){
    createDropDown();
    dg.querySelector(".column-display-selector-header").addEventListener("click", function(e){
        e.target.closest(".column-display-selector-container").classList.toggle("expand");
    });
    document.body.addEventListener("click", function(e){
        if (!e.target.closest(".column-display-selector-container")) {
            let allDD = document.querySelectorAll(".column-display-selector-container");
            for (let i=0;i<allDD.length;i++){
                allDD[i].classList.remove("expand");
            }
        }
    });
}

/*DropDown*/
function createDropDown() {
    let checkboxListContainer = document.createElement("div");
    checkboxListContainer.classList.add("column-display-selector-container");
    let dropDownHeader = document.createElement("div");
    dropDownHeader.classList.add("column-display-selector-header");
    let dropDownHeaderText = document.createElement("span");
    dropDownHeaderText.innerText = "Columns";
    dropDownHeader.appendChild(dropDownHeaderText);
    checkboxListContainer.appendChild(dropDownHeader);
    let checkboxList = document.createElement("div");
    checkboxList.classList.add("control-container", "check-box-list-container", "column-display-checkboxlist", "drag-sort-enable");
    let colOrder = 0;
    if (getDMValues(dg, "HasSelectableData")) colOrder = 1;
    for (let i = 0; i < arrColumnDefinitions.length; i++) {
        colOrder++;
        let labelText = arrColumnDefinitions[i].headerText;
        if (!labelText) labelText = arrColumnDefinitions[i].name;
        let display = arrColumnDefinitions[i].visible;
        let checkboxID = inputClass + "-id-" + arrColumnDefinitions[i].name;
        let checkboxContainer = document.createElement("div");
        checkboxContainer.classList.add("checkbox");
        checkboxContainer.setAttribute("order", colOrder);
        if (!display) checkboxContainer.style.display = "none";
        if (enableReorder) {
            let checkboxDrag = document.createElement("div");
            checkboxDrag.classList.add("dragger");
            checkboxContainer.appendChild(checkboxDrag);
        }
        let checkbox = document.createElement("input");
        checkbox.setAttribute("type", "checkbox");
        if (display) checkbox.checked = true;
        if (hideCols.indexOf(arrColumnDefinitions[i].name) > -1) {
            checkbox.checked = false;
            arrColumnDefinitions[i].visible = false;
        }
        checkbox.setAttribute("value", arrColumnDefinitions[i].name);
        checkbox.setAttribute("id", checkboxID);
        checkbox.addEventListener("change", toggleColumnDisplay);
        let checkboxLabel = document.createElement("label");
        checkboxLabel.setAttribute("for", checkboxID);
        checkboxLabel.innerText = labelText;
        checkboxContainer.appendChild(checkbox);
        checkboxContainer.appendChild(checkboxLabel);
        checkboxList.appendChild(checkboxContainer);
    }
    checkboxListContainer.appendChild(checkboxList);
    let dgHead = dg.querySelector(".data-grid-header");
    dgHead.appendChild(checkboxListContainer);
    if (enableReorder) (()=> {enableDragSort('drag-sort-enable');})();
}
function toggleColumnDisplay(e) {
    let el = e.target;
    let colIndex = Array.prototype.indexOf.call(el.closest(".column-display-checkboxlist").children, el.parentNode);
    let arrColsDefs = getDMValues(dg, "ColumnDefinitions");
    if (el.checked) {
        arrColsDefs[colIndex].visible = true;
    } else {
        arrColsDefs[colIndex].visible = false;
    }
    setDMValues(dg, "ColumnDefinitions", arrColsDefs);
    createCookie(pathname + "-" + inputClass + "-hidden-columns", getHiddenColsArray(), 30);
}
function getHiddenColsArray() {
    let hiddenCols = [];
    let arr = getDMValues(dg, "ColumnDefinitions");
    for (let i = 0; i < arr.length; i++) {
        if (!arr[i].visible) hiddenCols.push(arr[i].name);
    }
    return hiddenCols;
}
/*Reorder*/
function enableDragSort(listClass) {
    const sortableLists = dg.getElementsByClassName(listClass);
    Array.prototype.map.call(sortableLists, (list) => {enableDragList(list);});
}
function enableDragList(list) {
    Array.prototype.map.call(list.children, (item) => {enableDragItem(item);});
}
function enableDragItem(item) {
    item.setAttribute('draggable', true);
    item.ondrag = handleDrag;
    item.ondragend = handleDrop;
}
function handleDrag(item) {
    const selectedItem = item.target,
          list = selectedItem.parentNode,
          x = event.clientX,
          y = event.clientY;
    selectedItem.classList.add('drag-sort-active');
    let swapItem = document.elementFromPoint(x, y) === null ? selectedItem : document.elementFromPoint(x, y);
    if (list === swapItem.parentNode) {
        swapItem = swapItem !== selectedItem.nextSibling ? swapItem : swapItem.nextSibling;
        list.insertBefore(selectedItem, swapItem);
    }
}
function handleDrop(item) {
    let elmnt = item.target;
    elmnt.classList.remove('drag-sort-active');
    reorder(elmnt);
}
function reorder(el) {
    let colIndex = Array.prototype.indexOf.call(el.closest(".column-display-checkboxlist").children, el);
    let colName = el.querySelector('input').value;
    let arrColDefs = getDMValues(dg, "ColumnDefinitions");
    let prev = arrColDefs.findIndex(item => item.name === colName);
    arrColDefs = moveItem(arrColDefs, colIndex, prev);
    createCookie(pathname + "-" + inputClass + "-column-order", arrColDefs.map(a => a.name), 30);
}
function moveItem(array, to, from) {
    const item = array[from];
    array.splice(from, 1);
    array.splice(to, 0, item);
    return array;
}
/*Cookies*/
function createCookie(name, value, days) {
    let expires = "";
    if (days) {
        let date = new Date();
        date.setTime(date.getTime() + days * 24 * 60 * 60 * 1000);
        expires = "; expires=" + date.toGMTString();
    }
    document.cookie = name + "=" + value + expires + "; path=/";
}
function getCookie(c_name) {
    if (document.cookie.length > 0) {
        let c_start = document.cookie.indexOf(c_name + "=");
        if (c_start != -1) {
            c_start = c_start + c_name.length + 1;
            let c_end = document.cookie.indexOf(";", c_start);
            if (c_end == -1) {
                c_end = document.cookie.length;
            }
            return document.cookie.substring(c_start, c_end);
        }
    }
    return null;
}
/*DataModel*/
function getObjectName(obj) {
    let objname = obj.id.replace("-container","");
    do {
        let arrNameParts = objname.split(/_(.*)/s);
        objname = arrNameParts[1];
    } while ((objname.match(/_/g) || []).length > 0 && !scope[`${objname}Classes`]);
    return objname;
}
function getDMValues(ob, property) {
    let obname = getObjectName(ob);
    return scope[`${obname}${property}`];
}
function setDMValues(ob, property, value) {
    let obname = getObjectName(ob);
    scope[`${obname}${property}`] = value;
}
function getColItemIndex(name) { 
    return arrColumnDefinitions.findIndex(item => item.name === name);
}
```

## Page Setup
1. Drag a *DataGrid* control to the page ([see above](#database-connector-and-datagrid))
2. Add a class of your choosing to the *DataGrid* *Classes* property that uniquely identifies this DataGrid on this page (e.g datagrid-hide-cols)
3. Note: If multiple editable DataGrids are shown on one page, each DataGrid must have a unique classname

## Page.Load Event Setup
1. Populate your DataGrid with data ([see above](#database-connector-and-datagrid))
2. To initialise the DataGrid with hidden columns
   1. Drag a *List* action into the script (type: Any)
   2. Enter the column names you want to hide (as per the column properties shown in screenshot below)

List Value Example:
```json
= ["Healthy","Happy"]
```

![Column Name Property](images/ColumnName.png)

3. Drag the *ColumnDisplay* script into the script and complete the input parameters
   1. DataGridClass: The unique class you assigned to the *DataGrid* (e.g datagrid-hide-cols)
   2. InitialHiddenColumns: Leave blank or select your *List* containing the initial hidden columns from the dropdown
   3. EnableReordering: boolean to indicate whether users will be allowed to reorder DataGrid columns

## Applying the CSS
The CSS below is required for the correct functioning of the module. Some elements can be [customised](#customising-css) using a variables CSS file. 

**Stadium 6.6 or higher**
1. Create a folder called "CSS" inside of your Embedded Files in your application
2. Drag the two CSS files from this repo [*column-display-selector-variables.css*](column-display-selector-variables.css) and [*column-display-selector.css*](column-display-selector.css) into that folder
3. Paste the link tags below into the *head* property of your application
```html
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/column-display-selector.css">
<link rel="stylesheet" href="{EmbeddedFiles}/CSS/column-display-selector-variables.css">
``` 

![](images/ApplicationHeadProp.png)

**Versions lower than 6.6**
1. Copy the CSS from the two css files into the Stylesheet in your application

## Customising CSS
1. Open the CSS file called [*column-display-selector-variables.css*](column-display-selector-variables.css) from this repo
2. Adjust the variables in the *:root* element as you see fit
3. Overwrite the file in the CSS folder of your application with the customised file

## CSS Upgrading
To upgrade the CSS in this module, follow the [steps outlined in this repo](https://github.com/stadium-software/samples-upgrading)
