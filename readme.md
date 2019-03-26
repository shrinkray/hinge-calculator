# Hinge Calculator
My final project for the [Javascript for WP Bootcamp](https://www/javascriptforwp.com/bootcamp) is a refactored version of an existing script to evaluate form inputs and return a matching product. If no product will match, a response is given to tune the inputs. The data was engineered into a printed nomograph which is not very clear to most people. The solution is to provide an easy to use calculator instead of the document. 

Since the original calculator is more complicated than I anticipated, my purpose is to improve the form and modernize some of the existing JS. 

## Form 
The form takes six types of content. 
1. English or Metric Units
2. Door Height 
3. Door Width
4. Door Thickness
5. Door Weight
6. Door Material, either wood or metal

## Data
The previous app provides preconfigured information in the form of arrays. 
- Data is organized by weight, with smallest value at the top of the list
1. Door Weight Factor, structure of [weight, weight-coordinate]
2. Door Width Factor, structure [width, width-coordinate]
3. Minimum Thickness Range, based on English/metric conversion of door thicknesses
4. Hinge solutions 
5. Test Cases

## Functions



```javascript 
    // Input data validators

    function validate_height() {
      if (units_value() === "english") {
        lower = 4;
        upper = 240;
      }
      else {
        lower = 100;
        upper = 6000;
      }
      return validate_dimension(document.door_data.height, "Height", lower, upper, length_unit, 'height-error');
    }
```
  
```javascript 
    function validate_width() {
      if (units_value() === "english") {
        lower = 4;
        upper = 70;
      }
      else {
        lower = 100;
        upper = 1750;
      }
      return validate_dimension(document.door_data.width, "Width", lower, upper, length_unit, 'width-error');
    }
```

```javascript 
    function validate_thickness() {
      element = document.door_data.thickness;
      if(element.value === min_thickness[0].eng_value ) {
        document.getElementById('thickness-error').innerHTML = 'A minimum door thickness must be selected';
        invalid_input_field(element);
        return false;
      }
      document.getElementById('thickness-error').innerHTML = '';
      valid_input_field(element);
      return true;
    }
```
  

```javascript 
    function validate_weight() {
      if (units_value() === "english") {
        lower = 10;
        upper = 500;
      }
      else {
        lower = 5;
        upper = 225;
      }
      return validate_dimension(document.door_data.weight, "Weight", lower, upper, weight_unit, 'weight-error');
    }
```
  
```javascript 
    function validate_dimension(element, display, lower, upper, units, error_id) {
      let dimension = element.value;
      msg = "";

      if(is_empty(dimension)) {
        msg = display + " must be filled out"
      }
      else if(isNaN(dimension)) {
        msg = display + " must be a number"
      }
      else {
        let num = parseFloat(dimension);
        if((num < lower) || (num > upper)) {
          msg = display + " must be between " + lower.toString() + " and " + upper.toString() + " " + units;
        }
      }
      document.getElementById(error_id).innerHTML = msg;
      if(msg !== "") {
        invalid_input_field(element);
        return false;
      }
      valid_input_field(element);
      return true;
    }
```
  
```javascript 
    function calculate(table) {
      // Run all validations first
      // valid = validate_height();
      // valid &= validate_width();
      // valid &= validate_thickness();
      // valid &= validate_weight();
      // if(!valid) {
      //   return;
      // }

      // Lookup width coordinate
      let y1 = calculate_width_coordinate();

      // Lookup weight coordinate
      let y2 = calculate_weight_coordinate();

      // Calculate A line intersect
      let y3 = calculate_a_line_intersect(y1, y2);

      // Search solutions for matches
      for(let i = 0; i < solutions.length; i++) {
        check_possible_solution( solutions[i], y3 );
      }

      if(debug) {
        let row;
        let cell;
        row = table.insertRow();
        cell = row.insertCell();
        cell.innerHTML = "Width Y: " + y1.toFixed(2)
        cell = row.insertCell();
        cell.innerHTML = "Weight Y: " + y2.toFixed(2)
        cell = row.insertCell();
        cell.innerHTML = "A Line Y: " + y3.toFixed(2)
      }
    }
```

```javascript 
    function calculate_width_coordinate() {
      return table_interpolate(get_width(), door_width_factor);
    }

    function calculate_weight_coordinate() {
      return table_interpolate(get_weight(), door_weight_factor);
    }
```

```javascript 
    function table_interpolate(value, table) {
      let i;

      // Find index of high values; index of low values always one less.
      for(i = 1; (i < table.length) && (value > table[i][0]); i++)
        ;

      if( i >= table.length ) {
        return 0;
      }

      // Linear interpolate to get value (e.g. weight) coordinate wc
      //
      //  wc = value coordinate
      //  w = value
      //  i = index to high range values [w1, wc1]
      //  i-1 = index to low range values [w0, wc0]
      //
      //  (wc - wc0)/(wc1 - wc0) = (w - w0)/(w1 - w0)

      let w0  = table[i-1][0];
      let wc0 = table[i-1][1];
      let w1  = table[i][0];
      let wc1 = table[i][1];
      let wc = (((value - w0) * (wc1 - wc0)) / (w1 - w0)) + wc0;

      return wc;
    }

    function calculate_a_line_intersect(y1, y2) {
      //  x1 = 0
      //  x2 = 33.9 / 16
      //  x3 = 91.1 / 16
      //
      // y = mx + b
      // y3 = ((y2-y1/x2-x1))x2 + y1

      return (((y2 - y1)/(33.9)) * (91.1)) + y1;
    }

    function check_possible_solution( solution, intercept ) {
      // Check if within range
      // TODO Temp. adjustment to account for baseline not square
      if((intercept >= (solution.top_range + .4)) || (intercept < (solution.bot_range + .4))) {
        return;
      }
```

```javascript 
// Check if material is appropriate
      if((material_value() === "wood") && (solution.material !== "both")) {
        return;
      }


// Solution must have same or next size down minimum door thickess
      if(!check_for_thickness(solution, get_thickness())) {
        return;
      }
```

```javascript 
// Make sure at least one hinge for every 30 inches of door
      let needed = solution.hinges;
      needed = Math.max(needed, Math.ceil(get_height() / 30));

      solution_matches.push( solution_match = {model: solution.model, hinge_count: needed} );
    }
```

```javascript 
// Functions to access selection parameters, adjusted to English units.

    function get_height() {
      return convert_length_if_metric( height_value() );
    }

    function get_width() {
      return convert_length_if_metric( width_value() );
    }

    function get_thickness() {
      return thickness_value();
    }

    function get_weight() {
      let w = weight_value();
      if(units_value() === "metric") {
        w *= 2.2046;
      }
      return w;
    }


    function convert_length_if_metric( length ) {
      if(units_value() === "metric") {
        length *= 0.03937;
      }
      return length;
    }
```

```javascript 
    // Functions to pull raw values for calculation
    //   Note these are not used to get values for validation

    function height_value() {
      if(test) {
        return test_cases[test_num].height
      }
      else {
        return document.door_data.height.value;
      }
    }

    function width_value() {
      if(test) {
        return test_cases[test_num].width
      }
      else {
        return document.door_data.width.value;
      }
    }

    function thickness_value() {
      if(test) {
        return test_cases[test_num].thickness
      }
      else {
        return document.door_data.thickness.value;
      }
    }

    function weight_value() {
      if(test) {
        return test_cases[test_num].weight
      }
      else {
        return document.door_data.weight.value;
      }
    }

    function material_value() {
      if(test) {
        return test_cases[test_num].material
      }
      else {
        return document.door_data.material.value;
      }
    }

    function units_value() {
      if(test) {
        return test_cases[test_num].units
      }
      else {
        return measurement_system;
      }
    }
```

```javascript 
    // Calculate hinge solutions for input parameters
    function find_matches() {
      clear_results_display();

      let all_valid = true;
      all_valid &= validate_height();
      all_valid &= validate_width();
      all_valid &= validate_thickness();
      all_valid &= validate_weight();

      clear_solution_matches();

      if(!all_valid) {
        return;
      }

      set_status("Matching Soss Hinge Solutions:");

      calculate(table);

      display_solution_matches();
    }

    function check_for_thickness( solution, specified ) {
      for(let i=0; i<min_thickness.length; i++) {
        if(min_thickness[i].eng_value === specified) {
          if(in_array(solution.model, min_thickness[i].models)) {
            return true;
          }
        }
      }
      return false;
    }

    function in_array(value, list) {
      return (list.indexOf(value) > -1);
    }

    // Test execution
    function run_tests() {
      clear_results_display();
      set_status('Results for (' + test_cases.length + ') Test Cases:');

      for(test_num = 0; test_num < test_cases.length; test_num++) {
        solution_matches = [];

        display_test_case_header();

        calculate(table);

        display_solution_matches(test_cases[test_num].matches);
      }
    }

    // Display utilities

    function display_test_case_header() {
      let row;
      let cell;

      let test_case = test_num + 1;

      row = table.insertRow();
      cell = row.insertCell();
      cell.innerHTML = "TEST CASE #" + test_case;
      cell = row.insertCell();
      cell.innerHTML = "Height: " + test_cases[test_num].height;
      cell = row.insertCell();
      cell.innerHTML = "Width: " + test_cases[test_num].width;
      cell = row.insertCell();
      cell.innerHTML = "Thickness: " + test_cases[test_num].thickness;
      cell = row.insertCell();
      cell.innerHTML = "Weight: " + test_cases[test_num].weight;
      cell = row.insertCell();
      cell.innerHTML = "Material: " + test_cases[test_num].material;
      cell = row.insertCell();
      cell.innerHTML = "Units: " + test_cases[test_num].units;
    }

    function input_changing(id) {
      element = document.getElementById(id);
      element.value = "";
      valid_input_field(element);
      clear_solution_matches();
    }

    function clear_solution_matches() {
      solution_matches = [];
      clear_results_display();
      if(test) {
        set_status("Click button to run test suite.");
      }
      else {
        set_status("Enter door parameters then click to see solutions. For door thickness, measure the actual door thickness and select the largest thickness option that is not greater than the measured door thickness. Actual door thickness is often less than the thickness provided on the door specification.");
      }
    }

    function clear_results_display() {
      table = document.getElementById("results_display");
      while(table.childNodes[0]) {
        table.removeChild(table.childNodes[0]);
      }
    }

    function display_solution_matches(matches) {
      if(solution_matches.length === 0) {
        row = table.insertRow();
        cell = row.insertCell();
        cell.innerHTML = "No matches";
      }
      else
      {
        for(let i=0; i < solution_matches.length; i++) {
          row = table.insertRow();
          cell = row.insertCell();
          let msg = '';
          if( !matches ) {
            if( i === 0 ) {
              cell.className = 'recommended';
              msg = 'Soss recommends ';
            }
            else {
              cell.className = 'secondary';
              msg = 'You may also use ';
            }
          }
          cell.innerHTML = msg + "(" + solution_matches[i].hinge_count + ") Model #" + solution_matches[i].model + " hinges";
          if( matches && !solution_present(solution_matches[i], matches)) {
            cell = row.insertCell();
            cell.innerHTML = "- found.";
            row.style.color = "red";
          }
        }
      }

      if(matches) {
        for(let i=0; i<matches.length; i++) {
          if( !solution_present(matches[i], solution_matches)) {
            row = table.insertRow();
            cell = row.insertCell();
            cell.innerHTML = "(" + matches[i].hinge_count + ") Model #" + matches[i].model + " hinges";
            cell = row.insertCell();
            cell.innerHTML = "- not found.";
            row.style.color = "red";
          }
        }
      }
    }

    function solution_present( item, list) {
      for(let i=0; i < list.length; i++) {
        if( (item.model === list[i].model) && (item.hinge_count === list[i].hinge_count)) {
          return true;
        }
      }
      return false;
    }

    function set_status(msg) {
      document.getElementById('status_msg').innerHTML = msg;
    }

    // Form utility functions

    function initialize() {
      if(test) {
        document.door_data.style.display = "none";
      }
      else {
        document.getElementById("test_button").style.display = "none";
        measurement_system = "english";
      }
      set_units();
      clear_input();
    }

    function units_changed() {
      if(document.getElementsByName("units")[0].checked) {
        measurement_system = "english";
      }
      else {
        measurement_system = "metric";
      }
      clear_input();
      set_units();
    }

    function set_units() {
      if(units_value() === "english") {
        length_unit = "inches";
        weight_unit = "pounds";
      }
      else {
        length_unit = "millimeters";
        weight_unit = "kilograms";
      }

      document.getElementById('height-unit').innerHTML = length_unit;
      document.getElementById('width-unit').innerHTML = length_unit;
      document.getElementById('thickness-unit').innerHTML = length_unit;
      document.getElementById('weight-unit').innerHTML = weight_unit;

      select = document.getElementById('thickness-select');

      remove_options(select);

      for(let i=0; i<min_thickness.length; i++) {
        let option = document.createElement('option');
        option.value = min_thickness[i].eng_value;
        display = (units_value() === "english") ? min_thickness[i].eng_display : min_thickness[i].metric_display;
        option.innerHTML = display;
        select.appendChild(option);
      }
    }

    function remove_options(selectbox) {
      for(let i = selectbox.options.length - 1 ; i >= 0 ; i--) {
        selectbox.remove(i);
      }
    }

    function clear_input() {
      document.door_data.height.value = "";
      valid_input_field(document.door_data.height);
      document.door_data.width.value = "";
      valid_input_field(document.door_data.width);
      document.door_data.thickness.value = "0";
      valid_input_field(document.door_data.thickness);
      document.door_data.weight.value = "";
      valid_input_field(document.door_data.weight);
      document.getElementById('height-error').innerHTML = "";
      document.getElementById('width-error').innerHTML = "";
      document.getElementById('thickness-error').innerHTML = "";
      document.getElementById('weight-error').innerHTML = "";
      clear_solution_matches();
    }

    function invalid_input_field(element) {
      element.style.color = "red";
    }

    function valid_input_field(element) {
      element.style.color = "black";
    }

    function is_empty(str) {
      return str.replace(/^\s+|\s+$/gm,'').length === 0;
    }
```


```html 

<html>
    <body onload="initialize();">
        <div id="title_area">
        <h1>Soss Hinge Selector</h1>
        <!--  <h2>version 0.3.2</h2>-->
        </div>
        <hr>
        <form name="door_data">
        <table>
            <tr class="spaceUnder">
            <td><label for="units">Input Units</label></td>
            <td>&nbsp;</td>
            <td>
                <input id="units" type="radio" name="units" value="english" checked="" onclick="units_changed();">English (inches/pounds)</td>
            <td>&nbsp;</td>
            <td>
                <input id="units" type="radio" name="units" value="metric" onclick="units_changed();">Metric (millimeters/kilograms)</td>
            </tr>
        </table>
        <table>
            <tr class="spaceUnder">
            <td colspan="4">Enter Door Information:</td>
            </tr>
            <tr>
            <td><label for="height-value">Door Height: </label></td>
            <td>
                <input type="text" name="height" id="height-value" size="8" onfocus="input_changing(this.id);" onchange="validate_height();"></td>
            <td><span id="height-unit"></span></td>
            <td><span id="height-error" style="color: red"></span></td>
            </tr>
            <tr>
            <td><label for="width-value">Door Width: </label></td>
            <td>
                <input type="text" name="width" id="width-value" size="8" onfocus="input_changing(this.id);" onchange="validate_width();"></td>
            <td><span id="width-unit"></span></td>
            <td><span id="width-error" style="color: red"></span></td>
            </tr>
            <tr>
            <td><label for="thickness-select">Door Thickness: </label></td>
            <td>
                <select id="thickness-select" name="thickness" onfocus="input_changing(this.id);" onchange="validate_thickness();" ></select></td>
            <td><span id="thickness-unit"></span></td>
            <td><span id="thickness-error" style="color: red"></span></td>
            </tr>
            <tr>
            <td><label for="weight-value">Door Weight: </label></td>
            <td>

                <input type="text" name="weight" id="weight-value" size="8" onfocus="input_changing(this.id);" onchange="validate_weight();"></td>
            <td><span id="weight-unit"></span></td>
            <td><span id="weight-error" style="color: red"></span></td>
            </tr>
            <tr class="spaceUnder">
            <td><label for="material">Door Material: </label></td>
            <td colspan="2">
                <select id="material" name="material" onchange="clear_solution_matches();">
                <option value="wood" selected="selected">Wood</option>
                <option value="metal">Metal</option>
                </select>
            </td>
            </tr>
        </table>
        <button type="button" onclick="find_matches();">Calculate</button>
        </form>
        <button type="button" id="test_button" onclick="run_tests();">Run Tests</button>
        <div id="status_msg"></div>
        <table id="results_display"></table>
    </body>
</html>

```

