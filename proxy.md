     ex1:
        let user = {
            name: 'cyx',
            age: 25
        }
        let proxy = new Proxy(user, {
            get(target, property, context) {
                console.log(target, property, context);
                let value = target[property];
                if (!value) {
                    throw new Error(`${property}不存在`)
                }
                return value;
            }
        })
        let printUser = (property) => {
            console.log(`the user ${property} is ${proxy[property]}`)
        }
        printUser('name')
        printUser('na')



     ex2:   
         const api = new Proxy({}, {
            get(target, key, context) {
                console.log(target, key, context)
                return target[key] || ['get', 'post'].reduce((acc, key) => {
                    console.log(acc, key)
                    acc[key] = (config, data) => {
                        console.log(config, data)

                        if (!config && !config.url || config.url === '') throw new Error('Url cannot be empty.');
                        let isPost = key === 'post';

                        if (isPost && !data) throw new Error('Please provide data in JSON format when using POST request.');

                        config.headers = isPost ? Object.assign(config.headers || {}, { 'content-type': 'application/json;chartset=utf8' }) :
                            config.headers;

                        return new Promise((resolve, reject) => {
                            let xhr = new XMLHttpRequest();
                            xhr.open(key, config.url);
                            if (config.headers) {
                                Object.keys(config.headers).forEach((header) => {
                                    xhr.setRequestHeader(header, config.headers[header]);
                                });
                            }
                            xhr.onload = () => (xhr.status === 200 ? resolve : reject)(xhr);
                            xhr.onerror = () => reject(xhr);
                            xhr.send(isPost ? JSON.stringify(data) : null);
                        });
                    };
                    return acc;
                }, target)[key];
            },
            set() {
                throw new Error('API methods are readonly');
            },
            deleteProperty() {
                throw new Error('API methods cannot be deleted!');
            }
        });
        api.get({
            url: 'my-url',
            data: { a: 1 }
        }).then(
            (xhr) => {
                alert('Success');
            },
            (xhr) => {
                alert('Fail');
            }
        );