
Use [Tableau](https://www.tableau.com/) to query and visualize time series data
from {{< product-name >}}. Tableau supports multiple SQL dialects.

> Tableau is a visual analytics platform transforming the way we use data to
> solve problems—empowering people and organizations to make the most of their data.
>
> {{% cite %}}[tableau.com](https://www.tableau.com/why-tableau/what-is-tableau){{% /cite %}}

> [!Note]
> #### Tableau Desktop
> 
> These instructions are for **[Tableau Desktop](https://www.tableau.com/trial/tableau-software)**.
> Tableau Cloud and other Tableau products have not been tested, but may support
> connecting to {{< product-name >}} through the **Flight SQL JDBC driver**.

## Install Tableau Desktop

If you have not already, [download and install Tableau Desktop](https://www.tableau.com/products/desktop/download).

## Download and install the Flight SQL JDBC driver

To query {{< product-name >}} from Tableau, use the **Flight SQL protocol** and
the **Flight SQL JDBC driver**.

1.  **Download the Flight SQL JDBC driver.**
    1.  Visit the [Flight SQL JDBC driver](https://central.sonatype.com/artifact/org.apache.arrow/flight-sql-jdbc-driver/) page.
    2.  Select the **Versions** tab.
    3.  Click **Browse {{< icon "export" >}}** next to the version you want to
        download.
    4.  Click the `flight-sql-jdbc-driver-XX.XX.XX.jar` file
        (with only the `.jar` file extension) from the list of files
        to download the driver jar file.
        The version number in the file name is specific to the version you selected.

2.  **Copy the downloaded jar file into the following directory based on your operating system.**
  
    - **Windows**: `C:\Program Files\Tableau\Drivers`
    - **Mac**: `~/Library/Tableau/Drivers`
    - **Linux**: `/opt/tableau/tableau_driver/jdbc`

3.  **Start or restart Tableau.**

## Configure a JDBC server connection

1.  Open Tableau
2.  In the **Connect** column, under **To a Server**, select **Other Databases (JDBC)**.
    If that option isn't in the initial list, select **More...** to find it in
    the full list of connection options.
3.  Provide the required credentials:

    - **URL**: Your **InfluxDB cluster URL** with the following:

      - **Protocol**: `jdbc:arrow-flight-sql`
      - **Port**: `443`
      - **Query parameters**:
        - **useSystemTrustStore**: `false`

          > [!Note]
          > Setting `useSystemTrustStore=false` is only necessary on macOS and
          > doesn't actually affect the security of the connection to Tableau.
        
        - **database**: InfluxDB database name to query
    
      _See an [example connection URL](#example-connection-url)._
    
    - **Dialect**: PostgreSQL
    - **Username**: _Leave empty_
    - **Password**: _Leave empty_
    - **Properties File**: _Leave empty_

4.  Click **Sign In**.

#### Example connection URL

{{< code-placeholders "DATABASE_NAME" >}}
```
jdbc:arrow-flight-sql://{{< influxdb/host >}}?useSystemTrustStore=false&database=DATABASE_NAME
```
{{< /code-placeholders >}}

## Query InfluxDB

With the connection successfully established, query your time series data stored
in {{< product-name >}}. In the left pane:

1.  Under **Database**, select **public** from the drop-down menu.
2.  Under **Schema**, select **iox** from the drop-down menu.
3.  Under **Table**, click and drag the table you want to query into the query pane.
4.  Use Tableau to build and execute SQL queries.
    For more information, see the
    [Tableau Desktop documentation](https://help.tableau.com/current/pro/desktop/en-us/default.htm).
