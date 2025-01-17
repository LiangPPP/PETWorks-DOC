+++++++++++++++++++++++++++++++++++++++
k-匿名性去識別化 API
+++++++++++++++++++++++++++++++++++++++

以下程式碼旨在對資料進行去識別化以滿足 k-匿名性（k-anonymity）。更多 k-匿名性之說明，詳見 `此處 <#id4>`_ 。

我們以 ``data/adult.csv`` 作爲原始資料，``data/adult_hierarchy`` 目錄作爲資料階層定義，以及 ``attributeTypes`` 作為屬性型態定義，展示如何透過 PETWorks-framework 框架進行去識別化。

在以下程式碼中，我們透過 API ``PETAnonymization(originalData, tech, dataHierarchy, attributeTypes, maxSuppressionRate, k)``，以上述資料、“k-anonymity” 字串、屬性型態定義、最大抑制處理比率 maxSuppressionRate 、以及目標 k 值作爲參數，進行去識別化。

再來，我們透過 API ``report(result, path)``，以上述評估結果與輸出檔案位置 ``path`` 作爲參數，輸出去識別化結果。

範例程式碼: k-anonymization.py
------------------------------------

                                                           
.. code-block:: python
                                                           
  from PETWorks import PETAnonymization, output
  from PETWorks.attributetypes import *
  
  originalData = "data/adult.csv"
  dataHierarchy = "data/adult_hierarchy"
  
  attributeTypes = {
      "sex": QUASI_IDENTIFIER,
      "age": QUASI_IDENTIFIER,
      "workclass": QUASI_IDENTIFIER,
  }
  
  result = PETAnonymization(
      originalData,
      "k-anonymity",
      dataHierarchy,
      attributeTypes,
      maxSuppressionRate=0.6,
      k=6,
  )
  
  output(result, "output.csv")




輸出結果
---------------------------

上述程式碼將輸出滿足 k = 6 之 k-匿名性去識別化結果至 `output.csv`。檔案內容節錄如下：


+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| sex    | age | race | marital-status | education | native-country | workclass        | occupation | salary-class |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Male   | 37  | \*   | \*             | \*        | \*             | State-gov        | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Male   | 47  | \*   | \*             | \*        | \*             | Self-emp-not-inc | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Male   | 37  | \*   | \*             | \*        | \*             | Private          | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Male   | 52  | \*   | \*             | \*        | \*             | Private          | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Female | 27  | \*   | \*             | \*        | \*             | Private          | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Female | 37  | \*   | \*             | \*        | \*             | Private          | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Female | 47  | \*   | \*             | \*        | \*             | Private          | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Male   | 52  | \*   | \*             | \*        | \*             | Self-emp-not-inc | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| Female | 32  | \*   | \*             | \*        | \*             | Private          | \*         | \*           |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+
| ...    | ... | ...  | ...            | ...       | ...            | ...              | ...        | ...          |
+--------+-----+------+----------------+-----------+----------------+------------------+------------+--------------+

以本專案開發之 `k-匿名性再識別化工具 <https://petworks-doc.readthedocs.io/en/latest/kanonymity.html>`_ 進行檢測，可確認去識別化結果已滿足 k = 6 之 k-匿名性。

.. code-block:: json

  {
      "k": 6,
      "fulfill k-anonymity": true
  }


k-匿名性之定義
---------------------------

k-匿名性（k-anonymity）是一種指標，表示資料表中的任何一個資料列，其儲存之個體背景資訊皆與至少 k - 1 個資料列相同。其中，個體背景資訊爲可相互組合，識別出個體的隱私資訊，又稱準識別符（Quasi-identifier），由上方範例程式碼之屬性型態定義提供。

而 k 爲可調配之參數，數值介於 1 與無限大之間，數值越大，具相同個體背景資訊的資料列越多，識別出個體的難度越高。


去識別化之計算
---------------------------

本專案之 k-匿名性去識別化 API，為串接開源工具 ARX 之去識別化演算法進行。該演算法的處理流程可簡述如下：

1. 參考個體背景資訊定義與屬性型態定義，列舉並排序所有可能的去識別化處理強度組合。
2. 利用二分搜尋法，找出滿足 k 值設定，且去識別化處理強度最弱的組合。
3. 依據步驟 2. 的組合進行去識別化，輸出去識別化結果。

詳細處理流程請參閱 ARX 官方提供之 `去識別化演算法說明文件 <https://arx.deidentifier.org/development/algorithms/>`_ 。
