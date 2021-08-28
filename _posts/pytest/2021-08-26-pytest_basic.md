---
title: python test는 이제 pytest와 함께 - pytest tutorial (1)
date: 2021-08-26
tags: pytest test
category: programming
toc: true
--- 

python에는 JUnit에 영감을 받은 unittest가 있습니다. 그런데 unittest는 다소 장황한 문법으로 인해서 사용이 불편한 감이 있어요.
그래서 훨씬 간편한 pytest를 소개합니다.

* 이런 분께 추천합니다.
  * python에서 test를 하고 싶은 분
  * unittest의 class test 방식, 많은 assert variation에 지치신 분

## Install

```sh
(optional) python -m venv
(optional) source venv/bin/activate

pip install pytest
```

## Quick Start

```python
# test_first_step.py

def my_function(x):
    return x + 1


def test_my_function():
    assert my_function(3) == 4
    assert my_function(3) == 5  # 실패


class TestClass:
    def test_one(self):
        x = "this"
        assert "h" in x

    def test_two(self):
        x = "hello"
        assert hasattr(x, x)  # 실패
```

`pytest [file_name]` 으로 테스트 코드를 실행시킬 수 있습니다. 만약 파일명을 명시하지 않고 `pytest`만 실행하면 파일명에 `test_` prefix 가 붙어있는 파일의 테스트 코드를 검사합니다.

테스트 함수에는 `test_` prefix가 붙어있어야 합니다. `TestClass` 처럼 테스트 함수를 클래스로 묶어서 테스트도 가능한데, 테스트 클래스에는 `Test` prefix가 붙어있어야 합니다.

그럼 이제 pytest를 돌려봅니다.

```sh
$ pytest

=========================================================== test session starts ============================================================
platform darwin -- Python 3.8.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
rootdir: /Users/mskim/Developer/pytest_tutorial
collected 3 items                                                                                                                          

tests/test_first_step.py F.F                                                                                                         [100%]

================================================================= FAILURES =================================================================
_____________________________________________________________ test_my_function _____________________________________________________________

    def test_my_function():
        assert my_function(3) == 4
>       assert my_function(3) == 5
E       assert 4 == 5
E        +  where 4 = my_function(3)

tests/test_first_step.py:9: AssertionError
____________________________________________________________ TestClass.test_two ____________________________________________________________

self = <tests.test_first_step.TestClass object at 0x1040aba30>

    def test_two(self):
        x = "hello"
>       assert hasattr(x, x)
E       AssertionError: assert False
E        +  where False = hasattr('hello', 'hello')

tests/test_first_step.py:18: AssertionError
========================================================= short test summary info ==========================================================
FAILED tests/test_first_step.py::test_my_function - assert 4 == 5
FAILED tests/test_first_step.py::TestClass::test_two - AssertionError: assert False
======================================================= 2 failed, 1 passed in 0.06s ========================================================
(.venv)  ✘ mskim  ~/Developer/pytest_tutorial  
```

이렇게 어디에서 오류가 발생했는지 보여줍니다.

* 더 상세한 pytest 실행 방식을 보고싶다면: [pytest - usage](https://docs.pytest.org/en/6.2.x/usage.html)
* 더 많은 report 옵션을 보고싶다면: [pytest - detailed summary report](https://docs.pytest.org/en/6.2.x/usage.html#detailed-summary-report)

## Test Folder Structure

test 코드가 위 예시처럼 main코드와 섞여있으면 불편하겠죠. 그래서 보통은 아래처럼 tests 폴더 아래에 테스트 파일들을 몰아넣고, test 코드와 test할 코드명을 짝으로 만들어 놓습니다. `module/first_step.py` 와 `tests/module/test_first_step.py` 처럼요.

```sh
$ tree
.
├── module
│   ├── __init__.py
│   └── first_step.py
└── tests
    ├── __init__.py
    ├── module
    │   ├── __init__.py
    │   └── test_first_step.py
    └── test_tempfile.py

3 directories, 7 files
```

## Temporary directories and files

pytest는 `fixture` 라는 걸 제공하는데, 이걸 이용해서 여러 편리한 기능을 사용할 수 있습니다. 여기에서는 임시 폴더와 파일 경로를 받는 걸 정리합니다.

```python
# test_tempfile.py

def create_file(file_path, content):
    with open(file_path, 'w') as f:
        f.write(content)


def test_create_file(tmp_path):
    print(type(tmp_path), tmp_path)

    path = tmp_path / "sub"
    path.mkdir()

    file_path = path / "temp.txt"
    CONTENT = "content"
    create_file(file_path, CONTENT)
    
    assert file_path.read_text() == CONTENT  # CONTENT가 제대로 들어갔는지 확인
    assert len(list(tmp_path.iterdir())) == 1  # 디렉토리가 생성되었는지 확인
    assert 0    
```

위에서는 테스트 함수 인자가 없었지만, 여기에서는 `tmp_path` 를 받는 test 함수를 선언해줍니다. 그러면 테스트에 사용할 임시 경로를 `pathlib.Path` object로 받게 됩니다.
`tmp_path`를 이용해서 `create_file` 함수를 테스트했습니다.

위 파일을 테스트하면, 아래 결과가 나옵니다.

```sh
================================================================= FAILURES =================================================================
_____________________________________________________________ test_create_file _____________________________________________________________

tmp_path = PosixPath('/private/var/folders/5b/xqppj5qx5jdfn38_3nbgc9lm0000gn/T/pytest-of-mskim/pytest-46/test_create_file0')

    def test_create_file(tmp_path):
        print(type(tmp_path), tmp_path)
    
        path = tmp_path / "sub"
        path.mkdir()
    
        file_path = path / "temp.txt"
        CONTENT = "content"
        create_file(file_path, CONTENT)
    
        assert file_path.read_text() == CONTENT  # CONTENT가 제대로 들어갔는지 확인
        assert len(list(tmp_path.iterdir())) == 1  # 디렉토리가 생성되었는지 확인
>       assert 0
E       assert 0

tests/test_tempfile.py:25: AssertionError
----------------------------------------------------------- Captured stdout call -----------------------------------------------------------
<class 'pathlib.PosixPath'> /private/var/folders/5b/xqppj5qx5jdfn38_3nbgc9lm0000gn/T/pytest-of-mskim/pytest-46/test_create_file0
========================================================= short test summary info ==========================================================
FAILED tests/test_tempfile.py::test_create_file - assert 0
```

> **py.path vs. pathlib**  
> pytest는 os.path 의 interface인 py.path도 지원합니다. 다만, py 라이브러리는 pathlib 이 도입되면서 유지보수 모드가 되었습니다. 새로 시작하는 프로젝트라면 pathlib을 사용합시다.

---
