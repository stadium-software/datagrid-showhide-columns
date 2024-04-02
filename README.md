# DataGrid Show / Hide Columns

DataGrids sometimes contain a long list of fields and can get very wide. This repo contains a module that allows users to hide columns they don't want to see. Customisations to the column display are automatically saved per visitor and reapplied when the visitor returns. 

https://github.com/stadium-software/datagrid-showhide-columns/assets/2085324/bf547320-b77f-49bd-b03b-df9fa9ca2d6f

# Version 
1.1 Added logic to detect uniqueness of DataGrid class on page

1.2 Fixed Selectable column & hidden column bugs

1.3 Added column reordering feature; JS optimisations; CSS updates; Various bug fixes

## Application Setup
1. Check the *Enable Style Sheet* checkbox in the application properties

## Database, Connector and DataGrid
1. Use the instructions from [this repo](https://github.com/stadium-software/samples-database) to setup the database and DataGrid for this sample

## Global Script Setup
1. Create a Global Script called "ColumnDisplay"
2. Add two input parameters to the Global Script
   1. DataGridClass
   2. InitialHiddenColumns
3. Drag a *JavaScript* action into the script
4. Add the Javascript below into the JavaScript code property
```javascript
/* Stadium Script Version 1.3 https://github.com/stadium-software/datagrid-showhide-columns */
let dgClassName = "." + ~.Parameters.Input.DataGridClass;
let dg = document.querySelectorAll(dgClassName);
if (dg.length == 0) {
    dg = document.querySelector(".data-grid-container");
    dgClassName = ".data-grid-container";
} else if (dg.length > 1) {
    console.error("The class '" + dgClassName + "' is assigned to multiple DataGrids. DataGrids using this script must have unique classnames");
    return false;
} else { 
    dg = dg[0];
}
if (!dg) {
    console.error("No DataGrid was found on the page");
    return false;
}
let table = dg.querySelector("table");
let colsParameter = ~.Parameters.Input.InitialHiddenColumns;
const pathname = window.location.pathname;
let hideCols = [];
if (Array.isArray(colsParameter)) hideCols = colsParameter;
let cookieCols = getCookie(pathname + "-" + dgClassName + "-hidden-columns");
if (cookieCols !== null && cookieCols !== "") {
    hideCols = cookieCols.split(",");
} else if (cookieCols == "") {
    hideCols = [];
}
let arrColOrder = [];
let cookieColOrder = getCookie(pathname + "-" + dgClassName + "-column-order");
if (cookieColOrder !== null) {
    arrColOrder = cookieColOrder.split(",");
}
let options = {
        childList: true,
        subtree: true,
    },
    observer = new MutationObserver(initDataGridHide);
initDataGridHide();

function initDataGridHide(){
    if (table.querySelectorAll("tbody tr").length < 2) {
        observer.observe(table, options);
        return false;
    }
    observer.disconnect();
    let heads = formatHeadings();
    createDropDown(heads);
    for (let i=0;i<hideCols.length;i++) {
        setColumnVisibility(hideCols[i], false);
    }
    reorderList(arrColOrder);
    document.querySelector(".column-display-selector-header").addEventListener("click", function(e){
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
function createDropDown(arrHeads){
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
    for (let i = 0; i < arrHeads.length; i++) {
        let labelText = arrHeads[i].label;
        let colOrder = arrHeads[i].order;
        let display = arrHeads[i].display;
        let checkboxID = "id-" + colOrder;
        let checkboxContainer = document.createElement("div");
        checkboxContainer.classList.add("checkbox");
        checkboxContainer.setAttribute("order", colOrder);
        if (display) checkboxContainer.style.display = "none";
        let checkboxDrag = document.createElement("div");
        checkboxDrag.classList.add("dragger");
        let checkbox = document.createElement("input");
        checkbox.setAttribute("type", "checkbox");
        checkbox.checked = true;
        checkbox.setAttribute("value", labelText);
        checkbox.setAttribute("id", checkboxID);
        checkbox.addEventListener("change", toggleColumnDisplay);
        let checkboxLabel = document.createElement("label");
        checkboxLabel.setAttribute("for", checkboxID);
        checkboxLabel.innerText = labelText;
        checkboxContainer.appendChild(checkboxDrag);
        checkboxContainer.appendChild(checkbox);
        checkboxContainer.appendChild(checkboxLabel);
        checkboxList.appendChild(checkboxContainer);
    }
    checkboxListContainer.appendChild(checkboxList);
    let dgHead = document.querySelector(".data-grid-header");
    dgHead.appendChild(checkboxListContainer);
    (()=> {enableDragSort('drag-sort-enable');})();
}
function formatHeadings() {
    let headingElements = table.querySelectorAll("thead th");
    let headings = [];
    for (let i = 0; i < headingElements.length; i++) {
        let heading = {};
        headingElements[i].setAttribute("order", i + 1);
        heading.order = i + 1;
        if (headingElements[i].textContent) {
            heading.label = headingElements[i].textContent;
        } else if (headingElements[i].querySelector("input[type=checkbox")) {
            heading.label = "Checkboxes";
        } else {
            heading.label = "No Heading";
        }
        if (headingElements[i].style.display == "none") {
            heading.display = "none";
        }
        headings.push(heading);
    }
    return headings;
}
function toggleColumnDisplay(e) {
    let el = e.target;
    let o = el.parentNode.getAttribute("order");
    if (el.checked) {
        setColumnVisibility(o, true);
    } else {
        setColumnVisibility(o, false);
    }
}
function setColumnVisibility(no, visible) {
    dg.querySelector(".column-display-checkboxlist .checkbox[order='" + no + "'] input").checked = visible;
    let th = table.querySelector("th[order='" + no + "']");
    if (visible) {
        th.classList.remove("hidden-column");
    } else {
        th.classList.add("hidden-column");
    }
    let colNo = Array.prototype.indexOf.call(table.querySelector("thead tr").children, th) + 1;
    let cells = table.querySelectorAll("tbody tr td:nth-child(" + colNo + ")");
    for (let i = 0; i < cells.length; i++) {
        if (visible) {
            cells[i].classList.remove("hidden-column");
        } else {
            cells[i].classList.add("hidden-column");
        }
    }
    createCookie(pathname + "-" + dgClassName + "-hidden-columns", getHiddenColsArray(), 30);
}
function getHiddenColsArray() {
    let hiddenCols = [];
    let arr = table.querySelectorAll("th.hidden-column");
    if (arr) {
        for (let i = 0; i < arr.length; i++) {
            hiddenCols.push(arr[i].getAttribute("order"));
        }
    }
    return hiddenCols;
}
function getColOrderArray() {
    let colsOrder = [];
    let arr = table.querySelectorAll("thead th");
    if (arr) {
        for (let i = 0; i < arr.length; i++) {
            colsOrder.push(arr[i].getAttribute("order"));
        }
    }
    return colsOrder;
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
/*Reorder*/
function enableDragSort(listClass) {
    const sortableLists = document.getElementsByClassName(listClass);
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
    let th = dg.querySelector("thead tr th[order='" + el.getAttribute("order") + "']");
    let fromNo = Array.prototype.indexOf.call(th.parentNode.children, th) + 1;
    let dragSibling = el.previousSibling;
    let toNo = 1;
    let action = "beforebegin";
    if (dragSibling) {
        toNo = Array.prototype.indexOf.call(th.parentNode.children, dg.querySelector("thead tr th[order='" + dragSibling.getAttribute("order") + "']")) + 1;
        action = "afterend";
    }
    let siblingTh = dg.querySelector("thead tr th:nth-child(" + toNo + ")");
    siblingTh.insertAdjacentElement(action, th);
    let rows = dg.querySelectorAll("tbody tr");
    for (let i = 0; i < rows.length; i++) {
        let moveTd = rows[i].querySelector("td:nth-child(" + fromNo + ")");
        let siblingTd = rows[i].querySelector("td:nth-child(" + toNo + ")");
        siblingTd.insertAdjacentElement(action, moveTd);
    }
    createCookie(pathname + "-" + dgClassName + "-column-order", getColOrderArray(), 30);
}
function reorderList(list) {
    let parent = dg.querySelector(".column-display-checkboxlist");
    for (let i = 0; i < list.length; i++) {
        let el = parent.querySelector(".checkbox[order='" + list[i] + "']");
        parent.insertBefore(el, parent.children[i]);
        reorder(el);
    }
    return list;
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
   2. Enter the column numbers you want to hide (starting at 1 and including hidden columns)

List Value Example:
```json
= [2,5]
```

3. Drag the *ColumnDisplay* script into the script and complete the input parameters
   1. DataGridClass: The unique class you assigned to the *DataGrid* (e.g datagrid-hide-cols)
   2. InitialHiddenColumns: Leave blank or select your *List* containing the initial hidden columns from the dropdown

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
