[![Language grade: Python](https://img.shields.io/lgtm/grade/python/g/m-hu/small-exercise.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/m-hu/small-exercise/context:python)

# README

This exercise demonstrates a small http web server providing a RESTful API endpoint to calculate the sum of a list of integers in stream mode

* endpoints
    * /total/
        * method: post
        * payload: [1, 2, 3, 4]
            * a list of integers in json format. Other format or number type may cause an error message.
        * response: { "total": 10 }
            * a dictionary which contains a key named "total" whose value is the sum, in json format.

## Prerequisites

* In order to benefit from Tornado's full capability, please run it on Linux. Otherwise, please just use docker instead.
* The current implementation is tested against Python 3.6, which is normally defined by the embedded Dockerfile.

### Installation ###

* On a linux work station directly
    * Install python dependencies
        ```bash
        pip install -r requirements.txt
        ```
    * Install using setup.py
        ```bash
        python setup.py install
        ```
* Using container tool
    * Build docker image
        ```bash
        docker build -t sum_cal .
        ```

### How to use? ###

* Launch the web server after an installation with Python

    ```bash
    calsum --http-address=0.0.0.0 --app-debug=false --enable-auto-fork
    ```

* Launch it using docker-compose

    ```bash
    docker-compose -f docker-compose.yml build
    docker-compose -f docker-compose.yml up
    ```

* Test it with a small python snippet
  * submit a big calculation
    ```python
    import requests
    import json
    r = requests.post("http://localhost:8000/total/", data=json.dumps(list(range(10000001))))
    print(r.json())
    ```
    expected output
    ```json
    {'total': 50000005000000}
    ```
  * submit some calculation requests in parallel
    ```python
    import requests
    import json
    from multiprocessing import Pool
    
    
    def test_request(x):
        r = requests.post("http://localhost:8000/total/", data=json.dumps(list(range(100001+x))))
        return json.loads(r.content.decode()).get("total")
    
    def gen_result(x):
        return sum(range(100001 + x))
    
    with Pool(10) as p:
        expected = [gen_result(x) for x in range(100)]
        for e, r in zip(expected, p.map(test_request, [_ for _ in range(100)])):
            if e != r:
                raise Exception("{}!={}".format(e, r))
        print("ok at this scale")
    ```
    expected output
    ```bash
    ok at this scale
    ```

### How to develop? ###

* Install dependencies
    ```bash
    pip install -r requirements.txt
    ```

* Install in development mode and run the server
    ```bash
    python setup.py develop
    calsum --http-address=0.0.0.0 --app-debug=true
    ```

* Run tests
    ```bash
    python setup.py test
    ```
