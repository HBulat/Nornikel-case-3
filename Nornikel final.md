```python
#Программа работает на python 3.9.13 64-bit
```


```python
#Установка необходимых библиотек
```


```python
!pip install poppler-utils
```


```python
!pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```


```python
!pip install colpali-engine
```


```python
!pip install --upgrade byaldi
!pip install flash-attn
```


```python
!pip install pywin32
```


```python
folder = r'C:\Users\user\Desktop\Nornikel\data' #папка с файлами pdf
```


```python
#Код для получения метаданных с файлов
```


```python
import os
import win32com.client

def get_file_metadata(path, filename, metadata):
    try:
        if not os.path.isdir(path):
            raise ValueError(f"The specified path does not exist: {path}")
        
        sh = win32com.client.gencache.EnsureDispatch('Shell.Application', 0)
        ns = sh.NameSpace(path)

        if ns is None:
            raise RuntimeError(f"Failed to access the namespace for the path: {path}")

        file_metadata = dict()
        item = ns.ParseName(str(filename))

        if item is None:
            raise ValueError(f"The file does not exist in the specified path: {filename}")

        for ind, attribute in enumerate(metadata):
            attr_value = ns.GetDetailsOf(item, ind)
            if attr_value:
                file_metadata[attribute] = attr_value

        return file_metadata

    except Exception as e:
        print(f"Error retrieving metadata for {filename}: {e}")
        return {}

if __name__ == '__main__':
    metadata = []
    pdf_files = [f for f in os.listdir(folder) if f.endswith(".pdf")]

    for pdf_file in pdf_files:
        metadata1 = ['Name', 'Size', 'Item type', 'Date modified', 'Date created']
        file_metadata = get_file_metadata(folder, pdf_file, metadata1)
        metadata.append(file_metadata)
```


```python

```


```python
from byaldi import RAGMultiModalModel #загрузка модели RAG
RAG = RAGMultiModalModel.from_pretrained("vidore/colqwen2-v1.0")
```

    C:\Users\user\python 3913\lib\site-packages\tqdm\auto.py:21: TqdmWarning: IProgress not found. Please update jupyter and ipywidgets. See https://ipywidgets.readthedocs.io/en/stable/user_install.html
      from .autonotebook import tqdm as notebook_tqdm
    

    Verbosity is set to 1 (active). Pass verbose=0 to make quieter.
    

    `Qwen2VLRotaryEmbedding` can now be fully parameterized by passing the model config through the `config` argument. All other arguments will be removed in v4.46
    
    oading checkpoint shards: 100%|██████████| 2/2 [00:06<00:00,  3.24s/it]


```python
#Код для индексации pdf 
```


```python
#RAG.index(
    input_path="data", # Путь, где хранятся документы
    index_name='Nornikel', # # Имя, которое вы хотите дать своему индексу. Он будет сохранен по адресу `index_root/index_name/`
    store_collection_with_index=False, # Должен ли индекс хранить документы в кодировке base64
    doc_ids=[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24], # Id документов
    metadata=metadata, # Метаданные
    overwrite=True # Перезаписывать ли индекс, если он уже существует. Если False, то возвращается None и ничего не делается, если `index_root/index_name` уже существует
)
```


```python
#Код для загрузки индексов pdf 
```


```python
path_to_index = r"C:\Users\user\Desktop\Nornikel\.byaldi\Nornikel"
RAG = RAGMultiModalModel.from_index(path_to_index)
```

    Verbosity is set to 1 (active). Pass verbose=0 to make quieter.
    

    
    C:\Users\user\python 3913\lib\site-packages\byaldi\colpali.py:162: FutureWarning: You are using `torch.load` with `weights_only=False` (the current default value), which uses the default pickle module implicitly. It is possible to construct malicious pickle data which will execute arbitrary code during unpickling (See https://github.com/pytorch/pytorch/blob/main/SECURITY.md#untrusted-models for more details). In a future release, the default value for `weights_only` will be flipped to `True`. This limits the functions that could be executed during unpickling. Arbitrary objects will no longer be allowed to be loaded via this mode unless they are explicitly allowlisted by the user via `torch.serialization.add_safe_globals`. We recommend you start setting `weights_only=True` for any use case where you don't have full control of the loaded file. Please open an issue on GitHub for any issues related to this experimental feature.
      self.indexed_embeddings.extend(torch.load(file))
    


```python
#Функция для конвертации pdf в изображения  
```


```python
import os
from pdf2image import convert_from_path


def convert_pdfs_to_images(pdf_folder):
    pdf_files = [f for f in os.listdir(pdf_folder) if f.endswith(".pdf")]
    all_images = {}

    for doc_id, pdf_file in enumerate(pdf_files):
        pdf_path = os.path.join(pdf_folder, pdf_file)
        images = convert_from_path(pdf_path)
        all_images[doc_id] = images

    return all_images


all_images = convert_pdfs_to_images(folder)
```


```python
#Поиск с помощью RAG в pdf файлах 
```


```python
text_query = "Финансовые результаты НЛМК"
results = RAG.search(text_query, k=5)
results
```




    [{'doc_id': 19, 'page_num': 1, 'score': 15.5, 'metadata': {'Name': 'НЛМК 2024', 'Size': '942 КБ', 'Item type': 'Документ Adobe Acrobat', 'Date modified': '06.12.2024 17:34', 'Date created': '28.11.2024 1:43'}, 'base64': None},
     {'doc_id': 4, 'page_num': 28, 'score': 14.8125, 'metadata': {'Name': 'digital_production_5', 'Size': '12,2 МБ', 'Item type': 'Документ Adobe Acrobat', 'Date modified': '06.12.2024 17:34', 'Date created': '28.11.2024 1:43'}, 'base64': None},
     {'doc_id': 19, 'page_num': 4, 'score': 14.6875, 'metadata': {'Name': 'НЛМК 2024', 'Size': '942 КБ', 'Item type': 'Документ Adobe Acrobat', 'Date modified': '06.12.2024 17:34', 'Date created': '28.11.2024 1:43'}, 'base64': None},
     {'doc_id': 19, 'page_num': 2, 'score': 14.3125, 'metadata': {'Name': 'НЛМК 2024', 'Size': '942 КБ', 'Item type': 'Документ Adobe Acrobat', 'Date modified': '06.12.2024 17:34', 'Date created': '28.11.2024 1:43'}, 'base64': None},
     {'doc_id': 4, 'page_num': 10, 'score': 14.1875, 'metadata': {'Name': 'digital_production_5', 'Size': '12,2 МБ', 'Item type': 'Документ Adobe Acrobat', 'Date modified': '06.12.2024 17:34', 'Date created': '28.11.2024 1:43'}, 'base64': None}]




```python
#Поиск страниц из результатов
```


```python
def get_grouped_images(results, all_images):
    grouped_images = []

    for result in results:
        doc_id = result["doc_id"]
        page_num = result["page_num"]
        grouped_images.append(all_images[doc_id][page_num - 1]) 

    return grouped_images
```


```python
grouped_images = get_grouped_images(results, all_images)
```


```python
#Отображение результатов (страниц из pdf)
```


```python
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 5, figsize=(150, 100))

for i, ax in enumerate(axes.flat):
    img = grouped_images[i]
    ax.imshow(img)
    ax.axis("off")

plt.tight_layout()
plt.show()
```


    
![png](output_24_0.png)
    



```python

```
