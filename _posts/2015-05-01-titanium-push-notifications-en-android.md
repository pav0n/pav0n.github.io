---
layout: post
title: "Titanium Push Notifications en android"
description: ""
category: 
tags: [android,titanium,notifications]
---
En esta publicación tratare sobre el uso de notificaciones push utilizando titanium en android, antes de proseguir tengo que mencionar
que el modulo que se utilizara fue echo por [iamyellow](https://github.com/iamyellow) y mejorado por [DaikiSuganuma](https://github.com/DaikiSuganuma) 
y en el cual ahora he metido mi cuchara para compilarlo
con la versión 3.5.1.GA del sdk de titanium, también tengo que comentar que la parte del server la he desarrollado con python, solo mostrare
código y me limitare a explicar solo en donde crea que es necesario.

Primero instalamos el modulo [com.iamyellow.gcmjs](https://github.com/pav0n/gcm.js/blob/master/dist/net.iamyellow.gcmjs-android-0.6.zip?raw=true)

Agregamos la propiedad   GCM_sender_id con el numero de proyecto correspondiente para mas información consultar la [documentación de android](http://developer.android.com/google/gcm/gs.html).

{% highlight xml %}
<property name="GCM_sender_id" type="string">tu_numero</property>
{% endhighlight %}

En mi caso estoy usando alloy y estoy iniciando el modulo en el index.js pero en si no importa en donde se inicialice, lo pueden hacer 
en app.js para los que no utilicen alloy o en cualquier controller
{% highlight js %}
//index.js
var gcm = require('net.iamyellow.gcmjs');
//se agregan los eventos al módulos 
//success: se llamara cuando se obtenga una respuesta exitosa al registrarce para obtener el token.
gcm.addEventListener('success', function(ev) {
	Ti.API.info('token ' + ev.deviceToken );
	//aquí puedes enviar el token a el servidor o simplemente almacenarlo
	//a mi me gusta almacenarlo para despues validar si ya tengo el token o necesito lanzar el registro
	Ti.App.Properties.setString('token_android', ev.deviceToken); 
});
//error: se llamara cuando ocurre un error al registrarce
gcm.addEventListener('error', function (ev){
	alert(ev);
});
//callback: se llamara cuando llega una noficación en el dispositivo estando en primer plano
gcm.addEventListener('callback',function (data){
	alert(data);
});
var android_token = Ti.App.Properties.getString('token_android');
if(android_token){
	Ti.API.info("existe token " +  android_token);
}else{
	Ti.API.info("No existe token asi que se registra");
	gcm.registerForPushNotifications();	
}
{% endhighlight %}

Con esto ya se pueden recibir las notificaciones pero solo cuando se encuentra en primer plano para cuando nos encontramos en segundo
plano, cuando la aplicación se encuentra en segundo plano el modulo llamara el archivo gcm.js este debe ser ubicado a la misma altura del
directorio en donde se encuentra app.js esto si no se usa alloy, en mi caso como uso alloy lo coloque en assets/android/gcm.js 

Así quedo mi gcm.js basado en el post del blog de [iamyellow](http://iamyellow.net/post/40100981563/gcm-appcelerator-titanium-module) solo con
algunos cambios menores que comentare en el código.
{% highlight js %}
function s (service) {
	//obtenemos los datos que llegan en la notificación
	var serviceIntent = service.getIntent(),
	mJson = JSON.parse(serviceIntent.getStringExtra('data')),
	title = mJson.title ? mJson.title : '',
	statusBarMessage = mJson.message ? mJson.message : '',
	message = mJson.message ? mJson.message : '',
	notificationId = (function () {
		var str = '',
		now = new Date();
		var hours = now.getHours(),
		minutes = now.getMinutes(),
		seconds = now.getSeconds();
		str += (hours > 11 ? hours - 12 : hours) + '';
		str += minutes + '';
		str += seconds + '';
		var start = new Date(now.getFullYear(), 0, 0),
		diff = now - start,
		oneDay = 1000 * 60 * 60 * 24,
		day = Math.floor(diff / oneDay); // day has remaining days til end of the year
		str += day * (hours > 11 ? 2 : 1);
		var ml = (now.getMilliseconds() / 100) | 0;
		str += ml;
		return str | 0;
	})();
	//Lo principal  que se cambio del código que público iamyellow es que se elimino el
	//archivo gcm.activity.js y se cambio el clasname 'net.iamyellow.gcmjs.GcmjsActivity' por el nombre de la actividad principal
	//en mi caso es 'com.pav0n.notificationapp.NotificationAppActivity' que se forma entre con el id del proyecto,el
	//nombre de la aplicación agregándole la palabra Activity, en mi caso el id es com.pav0n.notificationapp y el 
	//nombre de la aplicación es NotificationApp  quedando 'com.pav0n.notificationapp.NotificationAppActivity'

	launcherIntent = Ti.Android.createIntent({
		className: 'com.pav0n.notificationapp.NotificationAppActivity',
		action: Ti.Android.ACTION_MAIN,
		packageName: 'com.pav0n.notificationapp',
		flags: Ti.Android.FLAG_ACTIVITY_NEW_TASK | Ti.Android.FLAG_ACTIVITY_SINGLE_TOP
	});
	launcherIntent.addCategory(Ti.Android.CATEGORY_LAUNCHER);
	// crear notificación
	var pintent = Ti.Android.createPendingIntent({
		intent: launcherIntent
	}),
	notification = Ti.Android.createNotification({
		contentIntent: pintent,
		contentTitle: title,
		contentText: message,
		tickerText: statusBarMessage,
		icon: Ti.App.Android.R.drawable.appicon,
		flags: Ti.Android.FLAG_AUTO_CANCEL | Ti.Android.FLAG_SHOW_LIGHTS,
		sound: Titanium.Android.NotificationManager.DEFAULT_SOUND
	});
	Ti.Android.NotificationManager.notify(notificationId, notification);
	Ti.Media.vibrate([0, 100, 100, 200, 100, 100, 200, 100, 100, 200]);
	service.stop();
}
{% endhighlight %}

Ahora esta es la porción de código que utilizo para enviar la notificación.
{% highlight python %}
# -*- coding: utf-8 -*-
import time
from gcm import GCM
REG_ID = 'APIKEY'
def send_android(not_register_callback, canonical_callback,tokens,data):
	"""tokens es una lista de tokens a los cual se enviara una notificación"""
    gcm = GCM(REG_ID)
    response = gcm.json_request(registration_ids=tokens,
                                data=data,collapse_key="%s"%int(round(time.time() * 1000)))
    if 'errors' in response:
        for error, reg_ids in response['errors'].items():
            if error in ['NotRegistered', 'InvalidRegistration']:
                for reg_id in reg_ids:
                    not_register_callback(reg_id)
    if 'canonical' in response:
        for reg_id, canonical_id in response['canonical'].items():
            canonical_callback(reg_id,canonical_id)
{% endhighlight %}

Eso es todo por el momento espero y a alguien le sirva
{% include JB/setup %}
