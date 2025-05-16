## A Complete Step-by-Step Guide
This guide will help you use **`react-gauge-component`** directly in a Python Dash app **without cloning any repository**.
### 1. Install npm and Node.js
**Why?** The `react-gauge-component` is a React-based library, which requires `npm` and Node.js for installation and bundling.
- Download and install Node.js from the [Node.js official website](https://nodejs.org/).
After installation, verify the versions of `npm` and `node` to confirm the setup

```bash
  node -v
  npm -v
```
### 2. Create a New Dash App and Directory Structure
- Create a new folder for your Dash project:
```bash
  mkdir dash-gauge-example
  cd dash-gauge-example 
```

- Initialize npm (to manage JavaScript dependencies):
```bash
  npm init -y
```
### 3. Install the `react-gauge-component` Library
- Install the **`react-gauge-component`** package along with required dependencies:

```bash
  npm install react react-dom prop-types react-gauge-component 
```
This ensures that you have the gauge component package and its required dependencies.

### 4. Create a Custom Dash Component Wrapper
To use `react-gauge-component` with Dash, you need to bundle the React library as a Dash-compatible component.
#### Folder Structure:
```text
dash-gauge-example/
├── src/
│   ├── lib/
│   │   └── GaugeComponent.js  # Custom React wrapper
│   ├── index.js               # Component entry point
├── app.py                     # Dash application file
├── package.json               # npm configuration (auto-created)
├── webpack.config.js          # Webpack bundling configuration
```

### 5. Create the React Wrapper (`GaugeComponent.js`)
In `src/lib/GaugeComponent.js`, write a simple wrapper for `react-gauge-component`:

```javascript
import React from 'react';
import PropTypes from 'prop-types';
import Gauge from 'react-gauge-component';

// Create a custom wrapper for Dash
const GaugeComponent = (props) => {
    const {
        value,
        min,
        max,
        label,
        colorMap,
        thickness,
        needleThickness,
        arcStart,
        arcEnd,
        width,
        height,
    } = props;

    // Allow size definition using percentages or fixed dimensions
    const gaugeStyle = {
        width: width,    // Can take percentage (e.g., '50%') or fixed value (e.g., '300px')
        height: height,  // Can take percentage (e.g., '50%') or fixed value (e.g., '300px')
    };

    return (
        <div style={gaugeStyle}>
            <Gauge
                value={value}
                minValue={min}
                maxValue={max}
                label={label}
                arcWidth={thickness}
                needleWidth={needleThickness}
                startAngle={arcStart}
                endAngle={arcEnd}
                segmentColors={Object.keys(colorMap).map((key) => colorMap[key])}
            />
        </div>
    );
};

GaugeComponent.propTypes = {
    value: PropTypes.number.isRequired,
    min: PropTypes.number,
    max: PropTypes.number,
    label: PropTypes.string,
    colorMap: PropTypes.object, // Map for dynamic colors, e.g., { 0: '#00ff00', 100: '#ff0000' }
    thickness: PropTypes.number,
    needleThickness: PropTypes.number,
    arcStart: PropTypes.number,
    arcEnd: PropTypes.number,
    width: PropTypes.string,  // Accept percentage (e.g., '100%') or fixed (e.g., '300px')
    height: PropTypes.string, // Accept percentage (e.g., '50%') or fixed (e.g., '200px')
};

// Default values for optional props
GaugeComponent.defaultProps = {
    min: 0,
    max: 100,
    label: '',
    colorMap: { 0: '#00ff00', 50: '#ffff00', 100: '#ff0000' },
    thickness: 10,
    needleThickness: 2,
    arcStart: 0,
    arcEnd: 180,
    width: '100%',   // Defaults to use full width
    height: '100%',  // Defaults to use full height
};

export default GaugeComponent;
```

### 6. Create the Component Entry Point (`src/index.js`)
In `src/index.js`, expose the Gauge wrapper for Dash to locate it:
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import GaugeComponent from './lib/GaugeComponent';

export { GaugeComponent };
```

### 7. Configure Webpack (`webpack.config.js`)
Set up **Webpack** to handle JavaScript modules and bundle the React component.
```javascript
const path = require('path');

module.exports = {
    entry: './src/index.js',
    mode: 'production',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
        library: 'react-gauge-component',
        libraryTarget: 'umd',
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                },
            },
        ],
    },
    resolve: {
        extensions: ['.js', '.jsx'],
    },
    externals: {
        react: 'React',
        'react-dom': 'ReactDOM',
    },
};
```
### 8. Build the React Component
- Run the Webpack bundler to generate the necessary bundle for Dash:
```shell
  npx webpack
```

### 9. Create the Dash Application (`app.py`)
Now you can use the custom Dash component in your application. Here’s an example:
```python
import dash
from dash import html, dcc
from dash.dependencies import Input, Output

app = dash.Dash(__name__)

# App Layout
app.layout = html.Div([
    html.H1("Dash Gauge Example"),
    dcc.Slider(id='gauge-slider', min=0, max=100, value=50),
    html.Div(id='gauge-output'),
], style={'textAlign': 'center'})

@app.callback(
    Output('gauge-output', 'children'),
    Input('gauge-slider', 'value')
)
def update_gauge(value):
    from react_gauge_component import GaugeComponent

    return GaugeComponent(
        value=value,
        min=0,
        max=100,
        label='Dynamic Size Example',
        colorMap={0: '#00FF00', 50: '#FFFF00', 100: '#FF0000'},
        thickness=15,
        needleThickness=4,
        arcStart=0,
        arcEnd=180,
        width='50%',  # New property: Gauge takes up 50% of its parent's width
        height='50%'  # New property: Gauge takes up 50% of its parent's height
    )


if __name__ == "__main__":
    app.run_server(debug=True)
```
### Quick Explanation of Steps:
1. Install `react-gauge-component` via npm — no cloning or Git access required.
2. Wrap the React library in a reusable Dash component (`GaugeComponent`).
3. Use Webpack to bundle the component into a file usable in Dash apps.
4. Use Python/Dash callbacks to dynamically update the component at runtime.
