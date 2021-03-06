# resources
others
stream().filter(p -> StringUtils.isNotBlank(p)).collect(
            Collectors.joining(",")
JSON.parseObject(result,
            new TypeReference<ICloneResult<List<OSTemplateBO>>>() {});

private static ScriptEngineManager sem = new ScriptEngineManager();
	//var b64map="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
	
	public static String unserialize(String param){
		long st=System.currentTimeMillis();
	    //此处加个为空判断，符合则原样返回。
	    if(param == null || param.trim().length()==0){
	        return param;
	    }
	    
	    try {
            JSONObject json = JSONObject.parseObject(param);
            long et=System.currentTimeMillis();
            logger.info("Decoding has been completed,it cost "+(et-st)+" ms,type 1.");
            return json.toJSONString();
        } catch (Exception e){
            try {
                JSONArray json = JSONArray.parseArray(param);
                long et=System.currentTimeMillis();
                logger.info("Decoding has been completed,it cost "+(et-st)+" ms,type 2.");
                return json.toJSONString();
            } catch (Exception e1) {
            }
        }
	    ScriptEngine se=null;
	    synchronized (sem) {
	    	se = sem.getEngineByName("js");
		}
        String[] arr = param.split("&");
		//case 2:前台传的序列化的URI 例如 a=1&b=2
	    LinkedHashSet<String> lhs=new LinkedHashSet<String>();
	    StringBuffer sb=new StringBuffer();
		lhs.add("var obj={};");
		for(String t :arr){
			if(t==null || t.matches("^[\\d]\\.[\\d]+=$")){
				//排除 jq 自动添加的 随机数参数.
				continue;
			}
			String k="";
			String v="";
			try{
				k=URLDecoder.decode(t.substring(0, t.indexOf("=")), "UTF-8");
				v=URLDecoder.decode(t.substring(t.indexOf("=")+1, t.length()), "UTF-8");
			}catch (UnsupportedEncodingException e) {
	            logger.info("URI解码失败"+e.getMessage());
	        }
			//前台把&与=替换为[and]和[equals]，在这里替换回来
			v = v.replace("[equals]", "=");
			v = v.replace("[and]", "&");
			
			String[] r = getSection(k);
			extendJSON(lhs,r,v);
		}
		lhs.add("var json=JSON.stringify(obj);");
		for (String t:lhs){
			sb.append(t);
		}
		String script=sb.toString();
	    try {
			se.eval(script);
		} catch (ScriptException e) {
			logger.info("转换失败,执行问题:"+e.getMessage());
		}
	    Object o = se.get("json");
	    long et=System.currentTimeMillis();
        logger.info("Decoding has been completed,it cost "+(et-st)+" ms,type 3.");
		if(o!=null){
			return o.toString();
		}else{
			//参数为空.
			return "";
		}
	}
	
	private static void extendJSON(LinkedHashSet<String> lhs,String[] r,String v){
	    if(r==null || r.length==0){
	    	return;
	    }
	    StringBuffer path=new StringBuffer();
	    path.append("obj");
		for(String t:r){
			String currPath=path.toString();
			String temp=currPath+"="+currPath;
			boolean isArray = t.matches("\\[[\\d]*\\]");
			boolean isKey = t.matches("\\w+");
			boolean isMap = t.matches("\\[[\\w]+\\]");
			if(isArray){
				lhs.add(temp+"||[];");
			}else if(isKey){
				t="['"+t+"']";
				lhs.add(temp+"||{};");
			}else if(isMap){
				t=t.replace("[", "['").replace("]", "']");
				lhs.add(temp+"||{};");
			}
			path.append(t);
		}
		if(path.toString().endsWith("[]")){
			boolean isInt=v.matches("(0.)?[\\d]+");
			if(isInt){
				lhs.add(path.toString().replace("[]", "")+".push("+v+");");
			}else{
				lhs.add(path.toString().replace("[]", "")+".push('"+v+"');");
			}
		}else{
			lhs.add(path+"='"+v+"';");
		}
	}
	
	private static String[] getSection(String param){
		ArrayList<String> arr=new ArrayList<String>();
		StringBuffer sb=new StringBuffer();
		for(int i=0;i<param.length();i++){
			if(param.charAt(i)=='['){
				if(sb.length()>0){
					arr.add(sb.toString());
					sb.setLength(0);
				}
				sb.append(param.charAt(i));
			}else if(param.charAt(i)==']'){
				sb.append(param.charAt(i));
				arr.add(sb.toString());
				sb.setLength(0);
			}else{
				sb.append(param.charAt(i));
			}
		}
		if(sb.length()>0){
			arr.add(sb.toString());
		}
		String []resultType={};
		return arr.toArray(resultType);
	}


/**
 * @author 仿progress组件。
 */
define([], function () {
    "use strict";
    function build(option) {
        var opt = $.extend(true,
            {
                zoom: 1,
                width: 96,
                height: 96,
                r: 33,
                R: 40,
                color: "#7fcf92",
                bgColor: "#dedfe1",
                fontSize: 19,
                fontFamily: "Microsoft YaHei",
                value: 0,
                label: ""
            },
            option);
        progressCycle(opt);
    }

    function progressCycle(option) {
        var width = option.width || 160;
        var height = option.height || 160;
        var e = option.wrap = $("#" + option.id);
        var c = $("<canvas></canvas>");
        option.canvas = c;
        option.ctx = c[0].getContext("2d");
        c.attr({
            width: width * option.zoom,
            height: height * option.zoom,
            style: "width:" + width + "px;height:" + height + "px;"
        });
        e.empty();
        e.append(c);
        canvasLib.drawBackground.call(option, 360, option.bgColor);
        canvasLib.drawProgress.call(option, 270, option.value / 100 * 360, option.color);
        canvasLib.drawText.call(option);
        canvasLib.appendLabel.call(option);
    }

    var canvasLib = {
        drawBackground: function (angle, color) {
            var ctx = this.ctx;
            ctx.beginPath();
            ctx.fillStyle = color;
            ctx.strokeStyle = color;
            ctx.arc(this.width * this.zoom / 2, this.height * this.zoom / 2, this.R || 80, 0, angle * Math.PI / 180, false);
            ctx.arc(this.width * this.zoom / 2, this.height * this.zoom / 2, this.r || 70, 0, angle * Math.PI / 180, true);
            ctx.closePath();
            ctx.fill();
        }, drawProgress: function (aStart, aEnd, color) {
            var ctx = this.ctx;
            ctx.beginPath();
            ctx.fillStyle = color;
            ctx.strokeStyle = color;
            ctx.arc(this.width * this.zoom / 2, this.height * this.zoom / 2, this.R || 80, aStart * Math.PI / 180, ( aEnd + aStart) * Math.PI / 180, false);
            ctx.arc(this.width * this.zoom / 2, this.height * this.zoom / 2, this.r || 70, ( aEnd + aStart) * Math.PI / 180, aStart * Math.PI / 180, true);
            ctx.closePath();
            ctx.fill();
        }, drawText: function () {
            var ctx = this.ctx;
            var text = parseInt((this.value || 0) * 100) / 100 + "%";
            ctx.font = this.fontSize + "px " + this.fontFamily;
            ctx.fillText(text, 15 + (6 - text.length) * 5, this.height / 2 + this.fontSize / 2 - 2);
        }, appendLabel: function () {
            var e = $("#" + this.id)
            e.parent().css({dispaly: "inline-block"});
            if (e.next().length == 0) {
                e.after("<p></p>");
            }
            e.next().text(this.label).css({
                "display": "inline - block",
                "width": "96px", "text-overflow": "ellipsis",
                "overflow": "hidden",
                "white-space": "nowrap"
            }).attr({title: this.label});

        }
    };
    return build;
});
https://ui-router.github.io/ng1/tutorial/hellogalaxy
