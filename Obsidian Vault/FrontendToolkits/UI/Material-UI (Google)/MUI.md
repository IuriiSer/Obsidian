[MUI](https://mui.com/material-ui/getting-started/installation/)
## How to use
### Common install
```
npm install @mui/material @emotion/react @emotion/styled

```
**Roboto font**
```
<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" />
```
**Font icons**
```
<link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons" />
```
**SVG icons**
```
npm install @mui/icons-material

```
### To use in Project
```
import { CssBaseline, Container } from '@mui/material';
import { createTheme, ThemeProvider } from '@mui/material/styles';
const theme = createTheme();

// ... func -> return (...)
<ThemeProvider theme={theme}>
	<CssBaseline>
		// YOUR App here
	</CssBaseline>
</ThemeProvider>
```