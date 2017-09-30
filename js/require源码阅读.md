1  require使用方法

在index.html 页面中使用script标签引入js：

 <script data-main="./index.js" src="./js/require.js"></script>

其中，data-main用来设置项目的入口

配置：

```javascript
require.config({

	paths:  {

		a: './js/a',

		b: './js/b',

		home: './js/home'

	}

	shim: {

		'home': {

			deps: ['a']

		},

		'a': {

			deps: ['b']

		}

	}

})
```

index.js：

```javascript
require(['home'], function (home) {

	console.log("index")

}
```

home.js:

```javascript
require(['a'], function (a) {

	console.log(a.add());

}
```

a.js:

```javascript
define(['b'], function(b){

	console.log("a");

	return {

		a: 2

		add: function(a){

			return this.a + b.b;

		}

	}

}）;
```

结果：

查看html元素,会在head中看到

<script src="./js/b.js"></script>

<script src="./js/a.js"></script>

<script src="./js/home.js"></script>

如果去掉require.config中的shim删除，那么程序就不知道js的依赖关系，此时查看head：

<script src="./js/home.js"></script>

<script src="./js/a.js"></script>

<script src="./js/b.js"></script>

由于没有了shim，程序就不知道js的依赖关系，只能通过执行js后的define的第一个参数中获得。

2  data-main 入口实现

```javascript
//寻找有data-main属性的script。
if (isBrowser && !cfg.skipDataMain) {
    //scripts()返回所有的script，逆向遍历这些script直到有一个func返回true
    eachReverse(scripts(), function (script) {
        //Set the 'head' where we can append children by
        //using the script's parent.
        if (!head) {
            head = script.parentNode;
        }   
		
		//查找data-main属性，来设置页面要加载的主script。如果找到，就将data-main的值设为baseUrl
        dataMain = script.getAttribute('data-main');
        if (dataMain) {
            mainScript = dataMain;

            //Set final baseUrl if there is not already an explicit one,
            //but only do so if the data-main value is not a loader plugin
            //module ID.
            //如多config没有设置baseUrl,并且data-main不包含！
            if (!cfg.baseUrl && mainScript.indexOf('!') === -1) {
                //如果data-main指向'./js/index.js',则baseUrl为./js/./,mainScript为index.js
                src = mainScript.split('/');
                mainScript = src.pop();
                subPath = src.length ? src.join('/')  + '/' : './';

                cfg.baseUrl = subPath;
            }

            //去掉mainScript中的.js
            mainScript = mainScript.replace(jsSuffixRegExp, '');

            //If mainScript is still a path, fall back to dataMain
            if (req.jsExtRegExp.test(mainScript)) {
                mainScript = dataMain;
            }
```


```javascript
            //把data-main指向的script放入到script中
            cfg.deps = cfg.deps ? cfg.deps.concat(mainScript) : [mainScript];

            return true;
        }
    });
}
```
3 define实现


```javascript
define = function (name, deps, callback) {
	var node, context;
    //匿名模块
    if (typeof name !== 'string') {
        //第一个参数调整为deps，第二个调整为callback
        callback = deps;
        deps = name;
        name = null;
    }

    //模块没有依赖
    if (!isArray(deps)) {
        callback = deps;
        deps = null;
    }

    //如果没有定义deps，并且callback为fuction，然后判断该模块是否是基于CommonJS规范的依赖
    if (!deps && isFunction(callback)) {
        deps = [];
        //删除callback中的注释，查找require引入的依赖
        if (callback.length) {
            callback
                .toString()
                .replace(commentRegExp, commentReplace)
                .replace(cjsRequireRegExp, function (match, dep) {
                    deps.push(dep);
                });

            deps = (callback.length === 1 ? ['require'] : ['require', 'exports', 'module']).concat(deps);
        }
    }

    //如果IE6-8有匿名define, 修正name和context的值
    if (useInteractive) {
        node = currentlyAddingScript || getInteractiveScript();
        if (node) {
            if (!name) {
                name = node.getAttribute('data-requiremodule');
            }
            context = contexts[node.getAttribute('data-requirecontext')];
        }
    }

    if (context) {
        context.defQueue.push([name, deps, callback]);
        context.defQueueMap[name] = true;
    } else {
        globalDefQueue.push([name, deps, callback]);
    }
};
```
4 加载js文件

req.load = function (context, moduleName, url) {

```javascript
    //创建script标签
    req.createNode = function (config, moduleName, url) {
        var node = config.xhtml ?
                document.createElementNS('http://www.w3.org/1999/xhtml', 'html:script') :
                document.createElement('script');
        node.type = config.scriptType || 'text/javascript';
        node.charset = 'utf-8';
        node.async = true;
        return node;
    };
    
    var config = (context && context.config) || {},
        node;
    if (isBrowser) {
        node = req.createNode(config, moduleName, url);
		//设置data-requirecontext和data-requiremodule，每个requirejs引入的js文件都包含这两个属性
        node.setAttribute('data-requirecontext', context.contextName);
        node.setAttribute('data-requiremodule', moduleName);

        if (node.attachEvent &&
                !(node.attachEvent.toString && node.attachEvent.toString().indexOf('[native code') < 0) &&
                !isOpera) {
            useInteractive = true;

            node.attachEvent('onreadystatechange', context.onScriptLoad);
        } else {
            node.addEventListener('load', context.onScriptLoad, false);
            node.addEventListener('error', context.onScriptError, false);
        }
        node.src = url;

        if (config.onNodeCreated) {
            config.onNodeCreated(node, config, moduleName, url);
        }
		//兼容IE6-8,script可能会在append之前执行，所以把node绑定在currentlyAddingScript中，在script添加后，将currentlyAddingScript设置为空
        currentlyAddingScript = node;
        if (baseElement) {
            head.insertBefore(node, baseElement);
        } else {
            head.appendChild(node);
        }
        currentlyAddingScript = null;

        return node;
    } else if (isWebWorker) {
        try {
            setTimeout(function() {}, 0);
            importScripts(url);

            //Account for anonymous modules
            context.completeLoad(moduleName);
        } catch (e) {
            context.onError(makeError('importscripts',
                            'importScripts failed for ' +
                                moduleName + ' at ' + url,
                            e,
                            [moduleName]));
        }
    }
};
```
当node加载完毕后，调用onScriptLoad

```javascript
 		onScriptLoad: function (evt) {
            if (evt.type === 'load' ||
                    (readyRegExp.test((evt.currentTarget || evt.srcElement).readyState))) 				{
                interactiveScript = null;

                //查找evt对应的script，提取data-requiremodule就知道module的name了。
                var data = getScriptData(evt);
                context.completeLoad(data.id);
            }
        },
```
completeLoad:

```javascript
        completeLoad: function (moduleName) {
                var found, args, mod,
                    shim = getOwn(config.shim, moduleName) || {},
                    shExports = shim.exports;
				//将全局的的globalDefQueue中的内容保存到defQueue中
                takeGlobalQueue();

                while (defQueue.length) {
                    args = defQueue.shift();
                    if (args[0] === null) {
                        args[0] = moduleName;
                        //如过当前的模块是匿名定义的，把当前的moduleName给它，并将found设为true
                        if (found) {
                            break;
                        }
                        found = true;
                    } else if (args[0] === moduleName) {
                        //Found matching define call for this script!
                        found = true;
                    }

					//实例化context.Module，并初始化
                    callGetModule(args);
                }
                context.defQueueMap = {};

                // 获取刚才实例化的Module对象
                mod = getOwn(registry, moduleName);

                if (!found && !hasProp(defined, moduleName) && mod && !mod.inited) {
                    if (config.enforceDefine && (!shExports || !getGlobal(shExports))) {
                        if (hasPathFallback(moduleName)) {
                            return;
                        } else {
                            return onError(makeError('nodefine',
                                             'No define call for ' + moduleName,
                                             null,
                                             [moduleName]));
                        }
                    } else {
                        //A script that does not call define(), so just simulate
                        //the call for it.
                        callGetModule([moduleName, (shim.deps || []), shim.exportsFn]);
                    }
                }

                checkLoaded();
            }
```
makeModuleMap：建立一个模块映射，其中包括模块的路径、父模块等属性

```javascript
function makeModuleMap(name, parentModuleMap, isNormalized, applyMap) {
            var url, pluginModule, suffix, nameParts,
                prefix = null,
                parentName = parentModuleMap ? parentModuleMap.name : null,
                originalName = name,
                isDefine = true,
                normalizedName = '';

            //给匿名模块定义一个内部名称
            if (!name) {
                isDefine = false;
                name = '_@r' + (requireCounter += 1);
            }

            nameParts = splitPrefix(name);
            prefix = nameParts[0];
            name = nameParts[1];

            if (prefix) {
                prefix = normalize(prefix, parentName, applyMap);
                pluginModule = getOwn(defined, prefix);
            }

            if (name) {
                if (prefix) {
                    if (isNormalized) {
                        normalizedName = name;
                    } else if (pluginModule && pluginModule.normalize) {
                        normalizedName = pluginModule.normalize(name, function (name) {
                            return normalize(name, parentName, applyMap);
                        });
                    } else {
                       
                        normalizedName = name.indexOf('!') === -1 ?
                                         normalize(name, parentName, applyMap) :
                                         name;
                    }
                } else {
                    normalizedName = normalize(name, parentName, applyMap);

                    nameParts = splitPrefix(normalizedName);
                    prefix = nameParts[0];
                    normalizedName = nameParts[1];
                    isNormalized = true;

                    url = context.nameToUrl(normalizedName);
                }
            }

            suffix = prefix && !pluginModule && !isNormalized ?
                     '_unnormalized' + (unnormalizedCounter += 1) :
                     '';

            return {
                prefix: prefix,
                name: normalizedName,
                parentMap: parentModuleMap,
                unnormalized: !!suffix,
                url: url,
                originalName: originalName,
                isDefine: isDefine,
                id: (prefix ?
                        prefix + '!' + normalizedName :
                        normalizedName) + suffix
            };
        }
```
require实现

    req = requirejs = function (deps, callback, errback, optional) {

```javascript
        //Find the right context, use default
        var context, config,
            contextName = defContextName;

        // 第一个参数不是模块依赖表达
        if (!isArray(deps) && typeof deps !== 'string') {
            // deps is a config object
            config = deps;
            if (isArray(callback)) {
                // 如果有依赖模块则调整参数, 第二个参数是deps
                deps = callback;
                callback = errback;
                errback = optional;
            } else {
                deps = [];
            }
        }

        if (config && config.context) {
            contextName = config.context;
        }

        context = getOwn(contexts, contextName);
        if (!context) {
            context = contexts[contextName] = req.s.newContext(contextName);
        }

        if (config) {
            context.configure(config);
        }

        return context.require(deps, callback, errback);
    };
```
 makeRequire

```javascript
makeRequire: function (relMap, options) {
                options = options || {};

                function localRequire(deps, callback, errback) {
                    var id, map, requireMod;

                    if (options.enableBuildCallback && callback && isFunction(callback)) {
                        callback.__requireJsBuild = true;
                    }

                    if (typeof deps === 'string') {
                        if (isFunction(callback)) {
                            //Invalid call
                            return onError(makeError('requireargs', 'Invalid require call'), errback);
                        }

                        //如果请求的是require|exports|module引入或导出的模块，通过handlers来操作						//这些模块
                        if (relMap && hasProp(handlers, deps)) {
                            return handlers[deps](registry[relMap.id]);
                        }

                        //Synchronous access to one module. If require.get is
                        //available (as in the Node adapter), prefer that.
                        if (req.get) {
                            return req.get(context, deps, relMap, localRequire);
                        }

                        // 通过require的模块名Normalize成需要的moduleMap对象
                        map = makeModuleMap(deps, relMap, false, true);
                        id = map.id;

                        if (!hasProp(defined, id)) {
                            return onError(makeError('notloaded', 'Module name "' +
                                        id +
                                        '" has not been loaded yet for context: ' +
                                        contextName +
                                        (relMap ? '' : '. Use require([])')));
                        }
                        return defined[id];
                    }

                    // 把globalQueue 转换到 context.defQueue，并把defQueue的每一个都实例化一个
                    // context.Module对象并初始化, 放入registry数组中表示可用.
                    intakeDefines();

                    //本次require结束后把所有deps标记为needing to be loaded.
                    context.nextTick(function () {
                        intakeDefines();

                        requireMod = getModule(makeModuleMap(null, relMap));

                        requireMod.skipMap = options.skipMap;

                        requireMod.init(deps, callback, errback, {
                            enabled: true
                        });

                        checkLoaded();
                    });

                    return localRequire;
                }

                mixin(localRequire, {
                    isBrowser: isBrowser,

                    toUrl: function (moduleNamePlusExt) {
                        var ext,
                            index = moduleNamePlusExt.lastIndexOf('.'),
                            segment = moduleNamePlusExt.split('/')[0],
                            isRelative = segment === '.' || segment === '..';

                        if (index !== -1 && (!isRelative || index > 1)) {
                            ext = moduleNamePlusExt.substring(index, moduleNamePlusExt.length);
                            moduleNamePlusExt = moduleNamePlusExt.substring(0, index);
                        }

                        return context.nameToUrl(normalize(moduleNamePlusExt,
                                                relMap && relMap.id, true), ext,  true);
                    },

                    defined: function (id) {
                        return hasProp(defined, makeModuleMap(id, relMap, false, true).id);
                    },

                    specified: function (id) {
                        id = makeModuleMap(id, relMap, false, true).id;
                        return hasProp(defined, id) || hasProp(registry, id);
                    }
                });

                if (!relMap) {
                    localRequire.undef = function (id) {
                        takeGlobalQueue();

                        var map = makeModuleMap(id, relMap, true),
                            mod = getOwn(registry, id);

                        mod.undefed = true;
                        removeScript(id);

                        delete defined[id];
                        delete urlFetched[map.url];
                        delete undefEvents[id];

                        eachReverse(defQueue, function(args, i) {
                            if (args[0] === id) {
                                defQueue.splice(i, 1);
                            }
                        });
                        delete context.defQueueMap[id];

                        if (mod) {
                            if (mod.events.defined) {
                                undefEvents[id] = mod.events;
                            }

                            cleanRegistry(id);
                        }
                    };
                }

                return localRequire;
            },            
```