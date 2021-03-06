Access to a custom data source is configured using the [CustomStore](/api-reference/30%20Data%20Layer/CustomStore '/Documentation/ApiReference/Data_Layer/CustomStore/') component. DevExtreme provides [ASP.NET](/concepts/05%20Widgets/List/03%20Data%20Binding/16%20Web%20API%20Service.md '/Documentation/Guide/Widgets/List/Data_Binding/Web_API_Service/') and [PHP](/concepts/05%20Widgets/List/03%20Data%20Binding/17%20PHP%20Service.md '/Documentation/Guide/Widgets/List/Data_Binding/PHP_Service/') extensions to configure the **CustomStore** and implement server-side data processing. You can also use the third-party extension for [MongoDB](/concepts/05%20Widgets/List/03%20Data%20Binding/18%20MongoDB%20Service.md '/Documentation/Guide/Widgets/List/Data_Binding/MongoDB_Service/'). 

You need to configure the **CustomStore** in detail for accessing a server built on another technology. Data in this situation can be processed on the client or server. In the former case, switch the **CustomStore** to the raw mode and load all data from the server in the [load](/api-reference/30%20Data%20Layer/CustomStore/1%20Configuration/load.md '/Documentation/ApiReference/Data_Layer/CustomStore/Configuration/#load') function as shown in the next example. 

#include common-code-customsource-rawmode-pagingdisabled

In the latter case, use the **CustomStore**'s **load** function to send data processing settings to the server. These settings are passed as a parameter to the **load** function. In case of the **Scheduler**, the only relevant setting is [filter](/api-reference/30%20Data%20Layer/CustomStore/LoadOptions/filter.md '/Documentation/ApiReference/Data_Layer/CustomStore/LoadOptions/#filter'). It is passed when the **Scheduler**'s [remoteFiltering](/api-reference/10%20UI%20Widgets/dxScheduler/1%20Configuration/remoteFiltering.md '/Documentation/ApiReference/UI_Widgets/dxScheduler/Configuration/#remoteFiltering') option is set to **true**.

After receiving this setting, the server should apply it to data and send back an object with the following structure:

    {
        data: [ ... ] // result data objects
    }

If the **Scheduler** allows a user to add, delete or update appointments, the **CustomStore** must implement the [insert](/api-reference/30%20Data%20Layer/CustomStore/1%20Configuration/insert.md '/Documentation/ApiReference/Data_Layer/CustomStore/Configuration/#insert'), [remove](/api-reference/30%20Data%20Layer/CustomStore/1%20Configuration/remove.md '/Documentation/ApiReference/Data_Layer/CustomStore/Configuration/#remove') and [update](/api-reference/30%20Data%20Layer/CustomStore/1%20Configuration/update.md '/Documentation/ApiReference/Data_Layer/CustomStore/Configuration/#update') operations as well. Here is a generalized configuration of the **CustomStore** for the **Scheduler** widget.

---

#####jQuery

    var schedulerDataSource = new DevExpress.data.DataSource({
        paginate: false,
        load: function(loadOptions) {
            var d = $.Deferred(),
                params = {};
            if("filter" in loadOptions && isNotEmpty(loadOptions["filter"])) 
                params[i] = JSON.stringify(loadOptions[i]);
            $.getJSON("http://mydomain.com/MyDataService", params)
                .done(function(result) {
                    // Here, you can perform operations unsupported by the server
                    // or any other operations on the retrieved data
                    d.resolve(result.data);
                });
            return d.promise();
        },
        insert: function(values) {
            return $.ajax({
                url: "http://mydomain.com/MyDataService/",
                method: "POST",
                data: values
            })
        },
        remove: function(key) {
            return $.ajax({
                url: "http://mydomain.com/MyDataService/" + encodeURIComponent(key),
                method: "DELETE",
            })
        },
        update: function(key, values) {
            return $.ajax({
                url: "http://mydomain.com/MyDataService/" + encodeURIComponent(key),
                method: "PUT",
                data: values
            })
        }
    });
    function isNotEmpty(value) {
        return value !== undefined && value !== null && value !== "";
    }
    $(function() {
        $("#schedulerContainer").dxScheduler({
            dataSource: schedulerDataSource,
            remoteFiltering: true
        });
    });

#####Angular

    <!--TypeScript-->
    import { ..., Inject } from "@angular/core";
    import { HttpClient, HttpClientModule, HttpParams } from "@angular/common/http";
    import { DxSchedulerModule } from "devextreme-angular";
    import DataSource from "devextreme/data/data_source";
    import CustomStore from "devextreme/data/custom_store";
    import "rxjs/add/operator/toPromise";
    // ...
    export class AppComponent {
        schedulerDataSource: any = {};
        constructor(@Inject(HttpClient) httpClient: HttpClient) {
            function isNotEmpty(value) {
                return value !== undefined && value !== null && value !== "";
            }
            this.schedulerDataSource = new DataSource({
                store: new CustomStore({
                    load: (loadOptions) => {
                        let params: HttpParams = new HttpParams();
                        if("filter" in loadOptions && isNotEmpty(loadOptions["filter"])) 
                            params = params.set(i, JSON.stringify(loadOptions[i]));
                        return httpClient.get("http://mydomain.com/MyDataService", { params: params })
                            .toPromise()
                            .then(result => {
                                // Here, you can perform operations unsupported by the server
                                // or any other operations on the retrieved data
                                return result.data;
                            });
                    },
                    insert: function(values) {
                        return httpClient.post('http://mydomain.com/MyDataService', values)
                            .toPromise();
                    },
                    remove: function(key) {
                        return httpClient.delete('http://mydomain.com/MyDataService' + encodeURIComponent(key))
                            .toPromise();
                    },
                    update: function(key, values) {
                        return httpClient.put('http://mydomain.com/MyDataService' + encodeURIComponent(key), values)
                            .toPromise();
                    }
                }),
                paginate: false
            });
        }
    }
    @NgModule({
        imports: [
            // ...
            DxSchedulerModule,
            HttpClientModule
        ],
        // ...
    })

    <!--HTML-->
    <dx-scheduler
        [dataSource]="schedulerDataSource"
        [remoteFiltering]="true">
    </dx-scheduler>

#####Vue

    <template>
        <dx-scheduler ... 
            :data-source="dataSource" />
    </template>
    <script>
    import DxScheduler from "devextreme-vue/scheduler";
    import CustomStore from "devextreme/data/custom_store";
    import DataSource from "devextreme/data/data_source";
    import 'whatwg-fetch';
    // ...
    function isNotEmpty(value) {
        return value !== undefined && value !== null && value !== "";
    }
    function handleErrors(response) {
        if (!response.ok)
            throw Error(response.statusText);
        return response;
    }
    const schedulerDataSource = new DataSource({
        store: new CustomStore({
            load: (loadOptions) => {
                let params = "?";
                if("filter" in loadOptions && isNotEmpty(loadOptions["filter"])) 
                    params = params.set(i, JSON.stringify(loadOptions[i]));
                params = params.slice(0, -1);
                return fetch(`https://mydomain.com/MyDataService${params}`)
                    .then(handleErrors)
                    .then(response => response.json())
                    .then((result) => {
                        // Here, you can perform operations unsupported by the server
                        // or any other operations on the retrieved data
                        return result.data;
                    });
            },
            insert: (values) => {
                return fetch("https://mydomain.com/MyDataService", {
                    method: "POST",
                    body: JSON.stringify(values),
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }).then(handleErrors);
            },
            remove: (key) => {
                return fetch(`https://mydomain.com/MyDataService/${encodeURIComponent(key)}`, {
                    method: "DELETE"
                }).then(handleErrors);
            },
            update: (key, values) => {
                return fetch(`https://mydomain.com/MyDataService/${encodeURIComponent(key)}`, {
                    method: "PUT",
                    body: JSON.stringify(values),
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }).then(handleErrors);
            }
        }),
        paginate: false
    })
    export default {
        // ...
        data() {
            return {
                dataSource: schedulerDataSource
            };
        },
        components: {
            // ...
            DxScheduler
        }
    }
    </script>

#####React

    import React from "react";
    import Scheduler from "devextreme-react/scheduler";
    import CustomStore from "devextreme/data/custom_store";
    import DataSource from "devextreme/data/data_source";
    import 'whatwg-fetch';
    // ...
    function isNotEmpty(value) {
        return value !== undefined && value !== null && value !== "";
    }
    function handleErrors(response) {
        if (!response.ok) 
            throw Error(response.statusText);
        return response;
    }
    const schedulerDataSource = {
        store: new CustomStore({
            load: (loadOptions) => {
                let params = "?";
                if("filter" in loadOptions && isNotEmpty(loadOptions["filter"])) 
                    params = params.set(i, JSON.stringify(loadOptions[i]));
                params = params.slice(0, -1);
                return fetch(`https://mydomain.com/MyDataService${params}`)
                    .then(handleErrors)
                    .then(response => response.json())
                    .then((result) => {
                        // Here, you can perform operations unsupported by the server
                        // or any other operations on the retrieved data
                        return result.data;
                    });
            },
            insert: (values) => {
                return fetch("https://mydomain.com/MyDataService", {
                    method: "POST",
                    body: JSON.stringify(values),
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }).then(handleErrors);
            },
            remove: (key) => {
                return fetch(`https://mydomain.com/MyDataService/${encodeURIComponent(key)}`, {
                    method: "DELETE"
                }).then(handleErrors);
            },
            update: (key, values) => {
                return fetch(`https://mydomain.com/MyDataService/${encodeURIComponent(key)}`, {
                    method: "PUT",
                    body: JSON.stringify(values),
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }).then(handleErrors);
            },
            paginate: false
        })
    }
    class App extends React.Component {
        render() {
            return (
                <Scheduler ...
                    dataSource={schedulerDataSource}>
                </Scheduler>
            );
        }
    }
    export default App;

---

#include common-demobutton with {
    url: "https://js.devexpress.com/Demos/WidgetsGallery/Demo/Scheduler/GoogleCalendarIntegration/"
}

#####See Also#####
- [Data Layer - DataSource Examples | Custom Sources](/concepts/30%20Data%20Layer/51%20Data%20Source%20Examples/3%20Custom%20Sources '/Documentation/Guide/Data_Layer/Data_Source_Examples/#Custom_Sources')
- [Data Layer - DataSource Examples | Connect to a RESTful Service](/concepts/30%20Data%20Layer/51%20Data%20Source%20Examples/3%20Custom%20Sources/0%20Connect%20to%20RESTful%20Service.md '/Documentation/Guide/Data_Layer/Data_Source_Examples/#Custom_Sources/Connect_to_RESTful_Service')
- [Data Layer - Overview](/concepts/30%20Data%20Layer/30%20Data%20Layer '/Documentation/Guide/Data_Layer/Data_Layer/')
- [Scheduler API Reference](/api-reference/10%20UI%20Widgets/dxScheduler '/Documentation/ApiReference/UI_Widgets/dxScheduler/')

[tags]scheduler, data binding, provide data, custom data source, CustomStore, DataSource, load, delete, add, update, remote filtering