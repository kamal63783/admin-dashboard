# Admin Dashboard

This application represents an admin dashboard UI with functionalities such as data table rendering, search, editing, deletion, pagination, and selection of rows.

## Table of Contents

- [Features](#features)
- [Technologies Used](#technologies-used)
- [Functionality](#functionality)
- [Installation](#installation)
- [Usage](#usage)
- [License](#license)

## Features

- **Search Bar:** Allows users to search for specific entries in the table.
- **Delete Selected Button:** Deletes the selected rows from the table.
- **Data Table:** Displays data in a table format, allowing editing, deletion, and row selection.
- **Selected Data Count:** Displays the count of selected rows.
- **Pagination Controls:** Provides navigation through multiple pages of data.

## Technologies Used

- **Nuxt.js:** Higher-level framework for Vue.js applications.
- **Vue.js:** Frontend framework for building user interfaces.
- **Axios:** A promise-based HTTP client for making API requests.
- **HTML/CSS:** Standard technologies for structuring and styling web content.
- **JavaScript (ES6+):** Programming language for frontend logic.

## Functionality

### Search Functionality

- Users can input search queries to filter data based on ID, name, email, or role.

### Data Table

- Each row displays information such as ID, name, email, role, and action buttons.
- Users can edit row details by clicking on the edit icon and save/cancel changes accordingly.
- Deletion of individual rows and selected rows is possible via respective buttons.
- Selection of rows can be performed using checkboxes.

### Pagination

- Facilitates pagination through multiple pages of data.
- Provides buttons for navigation: first, previous, next, last pages.
- Displays page information and allows users to jump to specific pages.

## Installation

To run this project locally, follow these steps:

1. **Clone the repository:**

   ```bash
   $ git clone https://www.github.com/kamal63783/admin-dashboard.git
   ```

2. **Navigate to directory:**

   ```bash
   $ cd admin-dashboard
   ```

3. **Install the dependencies:**

   ```bash
   $ npm install
   ```

## Usage

```bash
# Build the application for production
$ npm run build

# Launch the production server
$ npm run start

# Generate a static project
$ npm run generate

# Run the application in development mode
$ npm run dev
```

Open your browser and go to http://localhost:3000 to view the application.

Once the project is running locally, you can perform the following actions:

- Use the search bar to search for specific entries.
- Edit entries inline by clicking the edit icon and save or cancel changes accordingly.
- Delete individual entries or multiple selected entries using the delete functionality.
- Navigate through multiple pages using the pagination controls.

## License

This component is licensed under the [MIT License](LICENSE).

---

If you want to directly view the sample hosted application, navigate to https://admin-dashboard-kamal.vercel.app/. Open this link in desktop only.
