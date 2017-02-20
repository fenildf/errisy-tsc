# errisy-tsc
a customized typescript compiler transpiler

## why transpile?
In web devleopment, the communication between front end and back end have been an issue. Back end developers are normally required to write documentations for font end usage.

TypeScript has the advantage of applying dev-time type constrains in the codes to avoid tons of errors. TypeScript IDE in Visual Studio can also show the comments/descriptions for each class and function.

** Would it be fantastic if you back end classes can be transpiled into client classes? So you can CALL back end functions directly in your font end code **

## So we got an solution here:

Simple write your service as classes in *.rpc.ts files and extends from rpc.RPCService.

use **@rpc.service** to decorate the service class that you want to expose to the client.
use **@rpc.member** to decorate the member that you want to expose to the client.

**app.rpc.ts:**
```
/// <transpile path="C:\HTTP\npm\errisy-tsc\csclient.cs"/>
import * as rpc from 'errisy-rpc';
import { ImageItem } from './imageitem';
/**
 * the RPC service example.
 */
@rpc.service
export class AppService extends rpc.RPCService {
    /**
     * this is genarate a list of image slides for the front end to display
     */
    @rpc.member
    public async ImageList(): Promise<ImageItem[]> {
        return [
            new ImageItem('img/1.jpg', 300, 1000),
            new ImageItem('img/2.jpg', 500, 2000),
            new ImageItem('img/3.jpg', 200, 400),
            new ImageItem('img/4.jpg', 100, 800),
            new ImageItem('img/5.jpg', 300, 1400),
        ]
    }
}
```
So here is the transpiled client for Angular 2.
You can simply inject it into any Angular 2 component and use it to call it by await (because all rpc calls are wrapped in the async functions)
You can also invoke it by calling the [ImageList url] field, which is the link for it: "/app/app.rpc.js?AppService-ImageList&"

**Check out the magic of comments** The comments are also copied from the service to the client. That means the front end developers will see the output.

**app.rpc.ts:**
```
//Client file generated by RPC Compiler.
import { Injectable } from '@angular/core';
import { Http, Response } from '@angular/http';
import { Observable } from 'rxjs/Observable';
import { Observer } from 'rxjs/Observer';
import 'rxjs/add/operator/toPromise';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/catch';
import * as rpc from 'errisy-rpc';

import { ImageItem } from './imageitem';
/**
 * the RPC service example.
 */
@Injectable()
export class AppService {
	constructor(private $_Angular2HttpClient: Http){
	}
	/**Please set Base URL if this Remote Procedure Call is not sent to the default domain address.*/
	public $baseURL: string = "";
		/**
		 * this is genarate a list of image slides for the front end to display
		 */
		public ImageList(): Promise<ImageItem[]>{
			return this.$_Angular2HttpClient.post(this.$baseURL + '/app/app.rpc.js?AppService-ImageList', rpc.buildClientData()).map(rpc.Converter.convertJsonResponse).toPromise();
		}
		public get "ImageList url"():string{
			return this.$baseURL + "/app/app.rpc.js?AppService-ImageList&";
		}
}

```

## Transpiled C#:

C# can call the service as well with automatically generated front end client file! (For example, Xamarin project, Windows Desktop/WPF/Winform projects).
You must include the C# PolyFill for your C# front end.
Make sure you set the transpile path properly: 
**/// <transpile path="path in your computer"/>**
```CSharp
//Data Type Definition Generated by RPC compiler. Please do not modify this file.
namespace app
{
	/// <summary>
	/// the RPC service example.
	/// </summary>
	public class AppService
	{
		public AppService(string baseUrl){
			BaseUrl = baseUrl;
		}
		public string BaseUrl { get; set; }
		/// <summary>
		/// this is genarate a list of image slides for the front end to display
		/// </summary>
		public async System.Threading.Tasks.Task<ImageItem[]> ImageList()
		{
			return NodeJSRPC.Converter.convertJsonResponse<ImageItem[]>(await NodeJSRPC.HttpClient.Post(this.BaseUrl + "/app/app.rpc.js?AppService-ImageList", ));
		}
	}
}
```

#Can this handle file upload?
Yes! 
Use rpc.Polyfill.File to handle client side JavaScript File objects and rpc.Polyfill.Blob to handle client side JavaScript Blob objects.
This works for C# as well. Check it out yourself in the client files.

```
/// <transpile path="C:\HTTP\npm\errisy-tsc\csclient.cs"/>

import * as rpc from 'errisy-rpc';
import { ImageItem } from './imageitem';

/**
 * the RPC service example.
 */
@rpc.service
export class AppService extends rpc.RPCService {

    /**
     * this is genarate a list of image slides for the front end to display
     */
    @rpc.member
    public async ImageList(): Promise<ImageItem[]> {
        return [
            new ImageItem('img/1.jpg', 300, 1000),
            new ImageItem('img/2.jpg', 500, 2000),
            new ImageItem('img/3.jpg', 200, 400),
            new ImageItem('img/4.jpg', 100, 800),
            new ImageItem('img/5.jpg', 300, 1400),
        ]
    }
    /**
     * File upload
     * @param file
     */
    @rpc.member
    public async upload(file: rpc.Polyfill.File): Promise<string> {

    }
    /**
     * transfer of bytes
     * @param file
     */
    @rpc.member
    public async transfer(file: rpc.Polyfill.Blob): Promise<boolean> {

    }
}
```


##Work with errisy-server

errisy-server is an out-of-box http server written fully in the async/await middlewares.

##Warning of More Features:
I also made the TypeScript compiler to generate async methods that support cancel!
[TypeScript __awaiter support cancel!](https://github.com/Microsoft/TypeScript/issues/13854)

** In both front end and back end, you may have cases where you want to stop a Promise gracefully, but you can't do it with stardard Promise **

So here, without using RxJS, a few lines of codes in the __awaiter solved the cancel problem! In the errisy-tsc compiler, the __awaiter function returns a Task object instead of Promise (when you have npm module errisy-task available).

** Task object is cancellable and will append all internal Promises or Tasks in your async functions as its children and remove them when they are completed (remove to avoid error when you run animation script for ever) ** So when you cancel the task, it cancels all child tasks in the async function. That turns your async function very managable!


## MongoDB access
errisy-tsc also transpiles \*.data.ts to \*.sys.ts for database access.

er.data.ts

```
import * as rpc from 'errisy-rpc';

export interface IUser {
    _id: string;
    password: string;
    token: string;
}

@rpc.entity
export class User implements IUser {
    public _id: string;
    public password: string;
    public token: string;
}

/**
 * Data entity enables query
 */
@rpc.entity
export class Pet {
    public _id: string;
    /** index will enable automatic index set up in MongoDB for query optimization */
    @rpc.index.ascending
    public name: string;
    public images: string[];
    public body: PetBody;
}

/**
 * data struct is not a collection in mongoDB
 */
@rpc.struct
export class PetBody {
    public color: string;
    public weight: number;
}

/**
 * Session can be used to store any data, not just expireTime
 */
@rpc.entity
export class Session {
    public _id: string;
    public username: string;
    public expireTime: number;
}
```

The following data entity file is automatically transpiled for mongoDB access with type constrains
**You need [errisy-mongo](https://www.npmjs.com/package/errisy-mongo) for this feature**
[errisy-mongo](https://www.npmjs.com/package/errisy-mongo) defines all the mongoDB TypeScript wraps.
```
//Data Service file generate by RPC compiler.
//All your data definitions should be in the same *.data.ts file.
import { 
		EntityCollection, DataSet, NumberQuery, StringQuery, BooleanQuery, NumberArrayQuery, StringArrayQuery, BooleanArrayQuery, IObjectArrayQuery,
		IArrayQuery, IObjectQuery, IArrayProject, Aggregation
	} from 'errisy-mongo';
import { User, Pet, PetBody, Session } from './er.data';
import * as rpc from 'errisy-rpc';
export class UserQuery {
	public _id?: StringQuery;
	public password?: StringQuery;
	public token?: StringQuery;
	public $or?: UserQuery[];
	public $and?: UserQuery[];
}
export class UserCollection extends EntityCollection<User, UserQuery, UserProject> {
	public name: string = 'User';
}
export class UserProject {
	constructor() {
		let $dict: {[key: string]:any} = {};
	}
	public _id?: number;
	public password?: number;
	public token?: number;
}
export class PetQuery {
	public _id?: StringQuery;
	public name?: StringQuery;
	public images?: StringArrayQuery;
	public body?: PetBodyQuery;
	public $or?: PetQuery[];
	public $and?: PetQuery[];
}
export class PetCollection extends EntityCollection<Pet, PetQuery, PetProject> {
	public name: string = 'Pet';
}
export class PetProject {
	constructor() {
		let $dict: {[key: string]:any} = {};
		Object.defineProperty(this, 'body',
			{
				set: ($value: PetBodyProject|number) => {
					switch(typeof $value){
						case 'number':
							$dict['body'] = $value;
							break;
						default:
							for (let key in <any>$value) {
								this['body.' + key] = $value[key];
							}
							break;
					}
				},
				get: () => $dict['body']
			});
	}
	public _id?: number;
	public name?: number;
	public images?: number;
	public body?:PetBodyProject|number;
}
export class PetBodyQuery {
	public color?: StringQuery;
	public weight?: NumberQuery;
	public $or?: PetBodyQuery[];
	public $and?: PetBodyQuery[];
}
export class PetBodyProject {
	constructor() {
		let $dict: {[key: string]:any} = {};
	}
	public color?: number;
	public weight?: number;
}
export class SessionQuery {
	public _id?: StringQuery;
	public username?: StringQuery;
	public expireTime?: NumberQuery;
	public $or?: SessionQuery[];
	public $and?: SessionQuery[];
}
export class SessionCollection extends EntityCollection<Session, SessionQuery, SessionProject> {
	public name: string = 'Session';
}
export class SessionProject {
	constructor() {
		let $dict: {[key: string]:any} = {};
	}
	public _id?: number;
	public username?: number;
	public expireTime?: number;
}
export class erDataSet extends DataSet{
	public async User(suffix?: string): Promise<UserCollection> {
		if (!this.$db) await this.$connect();
		return new Promise<UserCollection>((resolve, reject) => {
			resolve(new UserCollection(this.$db, suffix));
		});
	}
	public async Pet(suffix?: string): Promise<PetCollection> {
		if (!this.$db) await this.$connect();
		return new Promise<PetCollection>((resolve, reject) => {
			resolve(new PetCollection(this.$db, suffix));
		});
	}
	public async Session(suffix?: string): Promise<SessionCollection> {
		if (!this.$db) await this.$connect();
		return new Promise<SessionCollection>((resolve, reject) => {
			resolve(new SessionCollection(this.$db, suffix));
		});
	}
	public get $Entities(): rpc.EntityDefinition[] {
		return [
			new rpc.EntityDefinition("User",[
				new rpc.FieldDefinition("_id", "string"),
				new rpc.FieldDefinition("password", "string"),
				new rpc.FieldDefinition("token", "string")
			]),
			new rpc.EntityDefinition("Pet",[
				new rpc.FieldDefinition("_id", "string"),
				new rpc.FieldDefinition("name", "string"),
				new rpc.FieldDefinition("images", "string[]"),
				new rpc.FieldDefinition("body", "PetBody")
			]),
			new rpc.EntityDefinition("Session",[
				new rpc.FieldDefinition("_id", "string"),
				new rpc.FieldDefinition("username", "string"),
				new rpc.FieldDefinition("expireTime", "number")
			])
		];
	}
	public async $Collection(definition: rpc.EntityDefinition, suffix?: string): Promise<EntityCollection<any, any, any>> {
		switch (definition.Name) {
			case 'User': return await this.User(suffix);
			case 'Pet': return await this.Pet(suffix);
			case 'Session': return await this.Session(suffix);
		}
	}
	public async $initializeIndices(): Promise<boolean>{
		await (await this.Pet()).createIndex({ name: 1 });
		return true;
	}
}

```
