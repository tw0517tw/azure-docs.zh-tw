---
title: 從 MySQL Workbench 連線到適用於 MariaDB 的 Azure 資料庫
description: 本快速入門提供的步驟，可以使用 MySQL Workbench 來連線及查詢來自適用於 MariaDB 的 Azure 資料庫的資料。
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.custom: mvc
ms.topic: quickstart
ms.date: 09/24/2018
ms.openlocfilehash: b3a4d00381361b5299e86b959d9775318ae81e88
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998222"
---
# <a name="azure-database-for-mariadb-use-mysql-workbench-to-connect-and-query-data"></a>適用於 MariaDB 的 Azure 資料庫︰使用 MySQL Workbench 來連線及查詢資料
本快速入門示範如何使用 MySQL Workbench 應用程式來連線到適用於 MariaDB 的 Azure 資料庫。 

## <a name="prerequisites"></a>必要條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [使用 Azure 入口網站建立適用於 MariaDB 的 Azure 資料庫伺服器](./quickstart-create-mariadb-server-database-using-azure-portal.md)
- [使用 Azure CLI 建立適用於 MariaDB 的 Azure 資料庫伺服器](./quickstart-create-mariadb-server-database-using-azure-cli.md)

## <a name="install-mysql-workbench"></a>安裝 MySQL Workbench
從 [MySQL 網站](https://dev.mysql.com/downloads/workbench/)下載 MySQL Workbench，並安裝在您的電腦上。

## <a name="get-connection-information"></a>取得連線資訊
取得連線到適用於 MariaDB 的 Azure 資料庫所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。

3. 按一下伺服器名稱。

4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![適用於 MariaDB 的 Azure 資料庫伺服器名稱](./media/connect-workbench/1_server-overview-name-login.png)

## <a name="connect-to-server-using-mysql-workbench"></a>使用 MySQL Workbench 連線到伺服器 
若要使用 MySQL Workbench 連線到適用於 MariaDB 的 Azure 資料庫伺服器，請使用下列步驟：

1.  在您的電腦上啟動 MySQL Workbench 應用程式。 

2.  在 [設定新連線] 對話方塊的 [參數] 索引標籤上輸入下列資訊︰

    ![設定新連線](./media/connect-workbench/2-setup-new-connection.png)

    | **設定** | **建議的值** | **欄位描述** |
    |---|---|---|
    |   連線名稱 | 示範連線 | 指定此連線的標籤。 |
    | 連線方式 | 標準 (TCP/IP) | 標準 (TCP/IP) 就足夠了。 |
    | 主機名稱 | 伺服器名稱 | 指定您稍早建立適用於 MariaDB 的 Azure 資料庫時所使用的伺服器名稱值。 示範的範例伺服器為 mydemoserver.mariadb.database.azure.com。 使用完整網域名稱 (\*.mariadb.database.azure.com)，如範例所示。 如果您不記得您的伺服器名稱，請依照上一節中的步驟執行，以取得連線資訊。  |
    | Port | 3306 | 連線到適用於 MariaDB 的 Azure 資料庫時一律使用連接埠 3306。 |
    | 使用者名稱 |  伺服器管理員登入名稱 | 輸入您稍早建立適用於 MariaDB 的 Azure 資料庫時所提供的伺服器管理員登入使用者名稱。 我們的範例使用者名稱為 myadmin@mydemoserver。 如果您不記得使用者名稱，請依照上一節中的步驟執行，以取得連線資訊。 格式為 *username@servername*。
    | 密碼 | 您的密碼 | 按一下 [儲存在保存庫...] 按鈕以儲存密碼。 |

3.   按一下 [測試連線] 以測試所有參數是否都已設定正確。 

4.   按一下 [確定] 可儲存連線。 

5.   在 [MySQL 連線] 清單中，按一下對應至您伺服器的圖格，然後等候建立連線。

        新的 SQL 索引標籤隨即開啟並出現空白的編輯器，可供您輸入查詢。
    
        > [!NOTE]
        > 根據預設，需要 SSL 連線安全性，而且會在適用於 MariaDB 的 Azure 資料庫伺服器上強制執行。 一般而言，您雖然不需要對 SSL 憑證進行其他設定，就能讓 MySQL Workbench 連線到您的伺服器，但還是建議您將 SSL CA 憑證繫結到 MySQL Workbench。 如果您需要停用 SSL，請前往 Azure 入口網站，然後按一下 [連線安全性] 頁面，以停用 [強制執行 SSL 連線] 切換按鈕。

## <a name="create-table-insert-read-update-and-delete-data"></a>建立資料表、插入、讀取、更新和刪除資料
1. 將範例 SQL 程式碼複製並貼到空白的 SQL 索引標籤，來說明某些範例資料。

    這段程式碼會建立名為 quickstartdb 的空白資料庫，然後會建立名為 inventory 的範例資料表。 它會插入一些資料列，然後讀取資料列。 它會使用 update 陳述式將資料進行變更，並再次讀取資料列。 最後會刪除資料列，然後再次讀取資料列。
    
    ```sql
    -- Create a database
    -- DROP DATABASE IF EXISTS quickstartdb;
    CREATE DATABASE quickstartdb;
    USE quickstartdb;
    
    -- Create a table and insert rows
    DROP TABLE IF EXISTS inventory;
    CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);
    INSERT INTO inventory (name, quantity) VALUES ('banana', 150);
    INSERT INTO inventory (name, quantity) VALUES ('orange', 154);
    INSERT INTO inventory (name, quantity) VALUES ('apple', 100);
    
    -- Read
    SELECT * FROM inventory;
    
    -- Update
    UPDATE inventory SET quantity = 200 WHERE id = 1;
    SELECT * FROM inventory;
    
    -- Delete
    DELETE FROM inventory WHERE id = 2;
    SELECT * FROM inventory;
    ```

    執行完畢後，螢幕擷取畫面會在 SQL Workbench 中顯示 SQL 程式碼的範例和輸出。
    
    ![執行範例 SQL 程式碼的 MySQL Workbench SQL 索引標籤](media/connect-workbench/3-workbench-sql-tab.png)

2. 若要執行範例 SQL 程式碼，請在 [SQL 檔案] 索引標籤的工具列中按一下閃電圖示。
3. 請注意頁面中間的 [結果方格] 區段中的三個索引標籤式結果。 
4. 請注意頁面底部的 [輸出] 清單。 隨即顯示每個命令的狀態。 

現在，您已使用 MySQL Workbench 連線到適用於 MariaDB 的 Azure 資料庫，並使用 SQL 語言查詢資料。

<!--
## Next steps
> [!div class="nextstepaction"]
> [Migrate your database using Export and Import](./concepts-migrate-import-export.md)
-->