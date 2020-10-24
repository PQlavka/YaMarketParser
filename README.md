{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "name": "YaM_Counter.ipynb",
      "provenance": [],
      "collapsed_sections": [],
      "authorship_tag": "ABX9TyMOqQ3vWNfXj7EAIEEtOEtG",
      "include_colab_link": true
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "view-in-github",
        "colab_type": "text"
      },
      "source": [
        "<a href=\"https://colab.research.google.com/github/PQlavka/YaMarketParser/blob/main/YaM_Counter.ipynb\" target=\"_parent\"><img src=\"https://colab.research.google.com/assets/colab-badge.svg\" alt=\"Open In Colab\"/></a>"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "f-RD4NI6SWVw"
      },
      "source": [
        "Виктор Остапюк\n"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "AjygEu-0GyTc"
      },
      "source": [
        "import requests\n",
        "from bs4 import BeautifulSoup\n",
        "from itertools import groupby\n",
        "\n",
        "def load_user_data(page, session):\n",
        "  # тут можно поменять фильтры\n",
        "  url = '%s&onstock=0&offer-shipping=pickup&delivery-interval=5&page=%d&local-offers-first=1' % (main_link, page)\n",
        "  request = session.get(url)\n",
        "  #print(url)\n",
        "  return request.text\n",
        "\n",
        "def contain_needed_obj(text):\n",
        "  soup = BeautifulSoup(text)\n",
        "  obj_list = soup.find('article', {'data-autotest-id': 'product-snippet'})\n",
        "  return obj_list is not None\n",
        "\n",
        "def not_end_of_list(text):\n",
        "  soup = BeautifulSoup(text)\n",
        "  endol = soup.find('a', {'aria-label': 'Следующая страница'})\n",
        "  return endol is not None\n",
        "\n",
        "def other_regions_span(text):\n",
        "  soup = BeautifulSoup(text)\n",
        "  span = soup.find('span', {'class': '_3J6m98nXNU'})\n",
        "  return span is not None\n",
        "\n",
        "def load_objects_names(dict, data):\n",
        "  for obj in data:\n",
        "    dict.append(obj.find('span', {'data-tid' : 'ce80a508'}).contents[0])\n"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "tpPcaeqMO_my"
      },
      "source": [
        "## **Парсер для Я.Маркет**\n",
        "* `unique` - `list[string]` - список уникальных элементов\n",
        "* `doubled` - `list[int]` - список индексов повторяющихся значений\n",
        "* `obj_dict` - `list[string]` - список всех элементов\n",
        "* `total` - `int` - общее кол-во элементов \n",
        "\n",
        "### Как использовать\n",
        "значение `main_link` заменить на свое (ссылка на каталог Я.Маркета должна оканчиваться атрибутом lr (lr=50))\n",
        "\n",
        "Cкопировать из браузера (\"Сеть\" в \"Инструментах разработчика\", самый продолжительный запрос -> заголовки запроса) значения User-Agent и Cookie в `s.headers.update(...)`\n",
        "\n",
        "Запустить всё по порядку (далее запускать только обновление `main_link` и применение парсера)\n",
        "\n",
        "Для выгрузки в excel использовать последний блок\n",
        "\n",
        "**Внимание!**\n",
        "\n",
        "Запросы рекомендую выполнять несколько раз (от 3). Парсер может дать осечку и выдать меньший результат, чем есть на деле (например: 120 вместо 670). Стоит работать с наибольшим результатом из 3-ех попыток. Причина мне пока не ясна :(\n"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "Qdz9ezcJiYSB"
      },
      "source": [
        "main_link = 'https://market.yandex.ru/catalog--mobilnye-telefony-v-permi/54726/list?hid=91491&lr=50'"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "u4Tn0xRBQjCX"
      },
      "source": [
        "s = requests.Session() "
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "XR5Hx0kCQlMV"
      },
      "source": [
        "# \"Сеть\" в \"Инструментах разработчика\", самый продолжительный запрос -> заголовки запроса\n",
        "s.headers.update({\n",
        "        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36',\n",
        "        'Cookie': 'yandexuid=9490329181603212206; yuidss=9490329181603212206; i=+mCagWm6/uNoWCwLQHp3+4gpyXpmMpXdMz0P6sSGd71DVDwjhxk68r/GdIiqOsQAAFWujM40GAADasUA4b78+zkevsM=; ymex=1918572206.yrts.1603212206#1918572206.yrtsi.1603212206; is_gdpr=0; cmp-merge=true; reviews-merge=true; mda=0; yandex_gid=213; is_gdpr_b=CMCAZxD/BygC; gdpr=0; _ym_uid=1603284610286524761; _ym_d=1603284625; zm=m-white_bender_zen-ssr.webp.css-https%3As3home-static_5tH4334hOcgqf3db37CyTmPLLbA%3Al; yc=1603543825.zen.cacS%3A1603288223; yabs-frequency=/5/0000000000000000/1cwmS9K0002eFK3xLB1mbG000AWz8EnKi72L0000g3qW/; ar=1603289130021461-829168; yp=1603371023.nwcst.1603287600_213_2#1605876608.ygu.1#1603889410.szm.1_25:1536x864:1536x754#1634821186.p_sw.1603285186#1603371797.ln_tp.01; ys=wprid.1603293331209667-697597603921457095042661-production-app-host-man-web-yp-275#bnrd.0899040111; skid=1840083581603352513; visits=1603284015-1603284015-1603352513; js=1; dcm=1; mOC=1; fonts-loaded=1; VS=0; ugcp=1; oMaSefD=1; oCnCPoS=1; pMaMeoG=1; regionChangeMethod=manual; _ym_isad=2; oMaPrfD=1; spravka=dD0xNjAzMzU4Mjg5O2k9MjEzLjg3LjQ1LjQ1O3U9MTYwMzM1ODI4OTYzNDY3Mzk0ODtoPTI3MDU4ZTA4YzQ5ZTRiMmIzOTA4NmJmZWJmZDdjMjBk; onstock=0; _ym_visorc=b; _ym_visorc_160656=b; yandexmarket=48%2CRUR%2C1%2C%2C%2C%2C2%2C0%2C0%2C50%2C0%2C0%2C12%2C0; lr=50; parent_reqid_seq=1603383349749%2F51654262a03f4c5f9f2700c744b20500%2C1603383418918%2Fcaee0341d35c9ce871971fcb44b20500%2C1603383790882%2F37b6da5af9dba0ae44504be144b20500%2C1603383887501%2Ff5be0575dc532a40be990de744b20500%2C1603384022006%2F9e3769956066928a50fb11ef44b20500; previousRegionId=43; previousRegionName=%D0%9A%D0%B0%D0%B7%D0%B0%D0%BD%D1%8C; currentRegionId=50; currentRegionName=%D0%9F%D0%B5%D1%80%D0%BC%D1%8C'\n",
        "    })"
      ],
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "syJQoWucje1E",
        "outputId": "14ff8c6f-cb1c-45bd-9eb7-ead34ad2aeb3",
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 67
        }
      },
      "source": [
        "# применение парсера\n",
        "obj_dict = [] # конечный результат в виде массива\n",
        "page = 1\n",
        "total = 0\n",
        "while True:\n",
        "  data = load_user_data(page, s)\n",
        "  if (contain_needed_obj(data) and not_end_of_list(data)):\n",
        "    soup = BeautifulSoup(data)\n",
        "    if other_regions_span(data):\n",
        "      span = soup.find('span', {'class': '_3J6m98nXNU'})\n",
        "      obj_list = span.find_all_previous('article', {'data-autotest-id': 'product-snippet'})\n",
        "      load_objects_names(obj_dict, obj_list)\n",
        "      total+=len(obj_list) \n",
        "      break\n",
        "  else:\n",
        "    obj_list = soup.find_all('article', {'data-autotest-id': 'product-snippet'})\n",
        "    load_objects_names(obj_dict, obj_list)\n",
        "    page+=1 \n",
        "    total+=len(obj_list) \n",
        "  else:\n",
        "    break\n",
        "\n",
        "doubled = [i for i, x in enumerate(obj_dict) if obj_dict.count(x) > 1] # повторяющиеся элементы\n",
        "unique = list(dict.fromkeys(obj_dict)) # уникальные элементы\n",
        "\n",
        "print(\"total: \", total)\n",
        "print(\"doubled: \", len(doubled))\n",
        "print(\"unique: \", len(unique))\n",
        "\n"
      ],
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "total:  475\n",
            "doubled:  112\n",
            "unique:  411\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "Artaw9zGO1aP"
      },
      "source": [
        "## **Выгрузка списка уникальных элементов в excel**"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "f4IL9ZCBKAmY"
      },
      "source": [
        "\n",
        "unique_sorted = sorted(unique)\n",
        "\n",
        "import pandas as pd\n",
        "\n",
        "df = pd.DataFrame() \n",
        "df['Name - YaM'] = unique_sorted[0::1] \n",
        "  \n",
        "# Converting to excel \n",
        "df.to_excel('result.xlsx', index = False) "
      ],
      "execution_count": null,
      "outputs": []
    }
  ]
}
