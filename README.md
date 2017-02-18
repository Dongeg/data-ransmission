# data-ransmission
前端向后端传输数据事例

```html
<!-- $Id: shipping_area_info.htm 16819 2009-11-25 06:21:17Z sxc_shop $ -->
{include file="pageheader.htm"}
{insert_scripts files="validator.js,../js/transport.js,../js/region.js,../js/jquery-3.1.1.min.js"}
<div class="main-div">
    <form method="post"  action="" name="theForm"  style="background:#FFF" enctype="multipart/form-data">
        <fieldset style="border:1px solid #DDEEF2">
            <table>
                <tr>
                    <td class="label">{$lang.shipping_area_name}:</td>
                    <td><input type="text" name="shipping_area_name" maxlength="60" size="30" value="{$shipping_area.shipping_area_name}" />{$lang.require_field}</td>
                </tr>
                <tr>
                    <td class="label">{$lang.fee_compute_mode}:</td>
                    <td>
                        <input type="radio"  disabled onclick="compute_mode('{$shipping_area.shipping_code}','weight')" name="fee_compute_mode" value="by_weight" />{$lang.fee_by_weight}
                        <input type="radio" checked="true"  onclick="compute_mode('{$shipping_area.shipping_code}','number')" name="fee_compute_mode" value="by_number" />{$lang.fee_by_number}
                    </td>
                </tr>
            </table>
            <div class="yunfei_ctn">
                <p id="add_rule_btn" onclick="addRule()">添加运费规则</p>
                <div class="add_transport">
                    <div class="add_transport_header">
                        <span>默认运费：</span><input type="text" id="default_num">件内，
                        <input type="text" id="default_fee">元，
                        每增加 <input type="text" id="default_add">件，
                        增加运费 <input type="text" id="default_add_fee">元
                    </div>
                    <div class="add_transport_main">
                        <table  border="1" cellspacing="0" cellpadding="0" style="border: 1px;"  >
                            <tr>
                                <td style="width: 50%;">运送到</td>
                                <td style="width: 10%;">首件（件）</td>
                                <td style="width: 10%;">首费（元）</td>
                                <td style="width: 10%;">续件（件）</td>
                                <td style="width: 10%;">续费（元）</td>
                                <td style="width: 10%;">操作</td>
                            </tr>
                            <tbody id="j_add_transport_main">


                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </fieldset>
        <div id="submit_btn">
            <input type="button" value="保存" id="all_submit">
        </div>

    </form>
<div id="select_ctn">
    <div id="select_province"  data-flag="">
            <table cellspacing="0" cellpadding="0" class="c_select">
                {foreach from=$region_area item=item}
                <tr>
                    {foreach from=$item item=region name=foo}
                    {if $smarty.foreach.foo.index eq 0}
                    <td><input type="checkbox" parent="{$region.region_area_id}" value="{$region.region_area_name}"  class="area">{$region.region_area_name}</td>
                    {/if}
                    <td><input type="checkbox"  parent="{$region.region_area_id}" value="{$region.region_name}" data-value="{$region.region_id}" class="province">{$region.region_name}</td>
                    {/foreach}
                </tr>
                {/foreach}
            </table>
            <input type="button" class="select_province_btn" id="province_submit" value="确定">
            <input type="button" class="select_province_btn" id="province_cancel" value="取消">
    </div>
</div>

</div>
<script language="JavaScript">
    <!--
    {literal}
    var now_index = 0;
    var times=[];
//    添加运费规则
    function addRule() {
        var timestamp = new Date().getTime();
        times.push(timestamp);
        var addData="<tr>" +
                "<td style='width: 50%;'>" +
                "<input type='hidden' data-type='"+timestamp+"' class='j_sheng'>" +
                "<div class='checkd_province'>" +
                "</div>" +
                "<div class='edit_province' data-id='"+timestamp+"' onclick='areaShow("+timestamp+")'>" +
                "编辑" +
                "</div>" +
                "</td>" +
                "<td style='width: 10%;'><input type='text' data-fir-num='"+timestamp+"' class='first_num'></td>" +
                "<td style='width: 10%;'><input type='text' data-fir-fee='"+timestamp+"'  class='first_fee'></td>" +
                "<td style='width: 10%;'><input type='text' data-add-num='"+timestamp+"' class='add_num'></td>" +
                "<td style='width: 10%;'><input type='text' data-add-fee='"+timestamp+"'  class='add_fee'></td>" +
                "<td style='width: 10%;' class='del_ruless' data-id='"+timestamp+"' onclick='del(this," + timestamp + ")'>删除</td>"+
                "</tr>"
        $("#j_add_transport_main").append(addData)
    }
//    确定省份
    $("#province_submit").bind('click',function(){
        select()
    })
//    删除
    function del(obj,index) {
        $(obj).parent().remove();
        $(".province").each(function(){
            if($(this).attr('bind_index') == index)
            {
                $(this).prop('checked',false);
                $(this).removeAttr('bind_index');
                $(this).removeAttr('disabled');
            }
        });
        $(".area").each(function(){
            if($(this).attr('bind_index') == index)
            {
                $(this).prop('checked',false);
                $(this).removeAttr('bind_index');
                $(this).removeAttr('disabled');
            }
        });
    }
//    编辑省份
    function areaShow(index){
        now_index = index;
        var area=$("#select_province");
        var quyu=$(".area")
        var province=$(".province");
        for(var i=0;i<province.length;i++){
            if(province[i].checked && $(province[i]).attr('bind_index') == index){
                $(province[i]).prop("checked",true);
                $(province[i]).removeAttr("disabled");
            }
        }

        for(var j=0;j<quyu.length;j++){
            if(quyu[j].checked && $(quyu[j]).attr('bind_index') == index){
                $(quyu[j]).prop("checked",true);
                $(quyu[j]).removeAttr("disabled");
            }
        }
        area.css('display','block');
        $("#select_ctn").css('display','block');
        area.attr('data-flag',index);
    }
//    判断全选按钮是否选中

        var province=$(".province");
        for(var i=0;i<province.length;i++){
            $(province[i]).bind('click',function(){
                if($(this).prop("checked")==false){
                    var par=$(this).attr('parent');
                    if($($(".area[parent='"+par+"']")[0]).prop("checked")==true){
                        $($(".area[parent='"+par+"']")[0]).prop("checked",false)
                        console.log(par)
                    }
                }
            })
        }


    //提交表单
    var allSubmit=$("#all_submit");
    allSubmit.bind('click',function () {
        var default_rule = [$("#default_num").val(),$("#default_fee").val(),$("#default_add").val(),$("#default_add_fee").val()];
        var len = times.length;
        var add_rule = [];
        for(var i = 0;i < len;i++){
            var temp = [];
            if(!($("input[data-type='"+times[i]+"']").val()==undefined)){
                temp.push($("input[data-type='"+times[i]+"']").val());
                temp.push($("input[data-fir-fee='"+times[i]+"']").val());
                temp.push($("input[data-fir-num='"+times[i]+"']").val());
                temp.push($("input[data-add-fee='"+times[i]+"']").val());
                temp.push($("input[data-add-num='"+times[i]+"']").val());
                add_rule.push(temp);
            }
        }
        console.log(default_rule);
        console.log(add_rule);
//        Ajax.call('url',{'def_rule':default_rule,'add_rule':add_rule},function(){}, 'POST', 'JSON');
    })

    function select() {
        var data=[];
        var dataID=[];
        var quyu=$(".area");
        var province=$(".province");
        for(var i=0;i<province.length;i++){
            if(province[i].checked && !$(province[i]).attr('disabled')){
                $(province[i]).prop("checked",true);
                $(province[i]).attr("disabled","disabled");
                $(province[i]).attr('bind_index',now_index);
                data.push($(province[i]).attr('value'));
                dataID.push($(province[i]).attr('data-value'))
            }
        }
        for(var j=0;j<quyu.length;j++){
            if(quyu[j].checked){
                $(quyu[j]).prop("checked",true);
                $(quyu[j]).attr("disabled","disabled");
                $(quyu[j]).attr('bind_index',now_index);
            }
        }

        console.log(data)
        var dataId=$("#select_province").attr('data-flag')
        $($("div[data-id='"+dataId+"']").prev()).text(data);
        $("input[data-type='"+now_index+"']").val(dataID);
        $("#select_province").css('display','none');
        $("#select_ctn").css('display','none');
        return false;
    }
    //多选
    var area=$(".area");
    var province=$(".province");
    for(var i=0;i<area.length;i++){
        area[i].index=i;
        $(area[i]).bind('click',function(){
            var par=$(this).attr('parent')
            if(this.checked){
                $("input[parent='"+par+"']").prop("checked", true);
            }
            else{
                $("input[parent='"+par+"']").prop("checked", false);
            }
        })
    }
    $("#province_cancel").bind('click',function(){
        var area=$("#select_province")
        area.css('display','none')
        $("#select_ctn").css('display','none')
        return false
    })
    region.isAdmin = true;
    onload = function()
    {
        $(".province").each(function(){
            $(this).prop('checked',false);
            $(this).removeAttr('bind_index');
            $(this).removeAttr('disabled');
        });
        $(".area").each(function(){
            $(this).prop('checked',false);
            $(this).removeAttr('bind_index');
            $(this).removeAttr('disabled');
        });
        document.forms['theForm'].elements['shipping_area_name'].focus();

        var selCountry = document.forms['theForm'].elements['country'];
        if (selCountry.selectedIndex <= 0)
        {
            selCountry.selectedIndex = 0;
        }

        region.loadProvinces(selCountry.options[selCountry.selectedIndex].value);

        // 开始检查订单
        startCheckOrder();
    }

    /**
     * 检查表单输入的数据
     */
    function validate()
    {
        validator = new Validator("theForm");

        validator.required('shipping_area_name', no_area_name);
        validator.isInt('free_money', invalid_free_mondy, true);

        var regions_chk_cnt = 0;
        for (i=0; i<document.getElementsByName('regions[]').length; i++)
        {
            if (document.getElementsByName('regions[]')[i].checked == true)
            {
                regions_chk_cnt++;
            }
        }

        if (regions_chk_cnt == 0)
        {
            validator.addErrorMsg(blank_shipping_area);
        }

        return validator.passed();
    }
    /**
     * 添加一个区域
     */
    function addRegion()
    {
        var selCountry  = document.forms['theForm'].elements['country'];
        var selProvince = document.forms['theForm'].elements['province'];
        var selCity     = document.forms['theForm'].elements['city'];
        var regionCell  = document.getElementById("regionCell");


        if (selCity.selectedIndex > 0)
        {
            regionId = selCity.options[selCity.selectedIndex].value;
            regionName = selCity.options[selCity.selectedIndex].text;
        }
        else
        {
            if (selProvince.selectedIndex > 0)
            {
                regionId = selProvince.options[selProvince.selectedIndex].value;
                regionName = selProvince.options[selProvince.selectedIndex].text;
            }
            else
            {
                if (selCountry.selectedIndex >= 0)
                {
                    regionId = selCountry.options[selCountry.selectedIndex].value;
                    regionName = selCountry.options[selCountry.selectedIndex].text;
                }
                else
                {
                    return;
                }
            }
        }

        // 检查该地区是否已经存在
        exists = false;
        for (i = 0; i < document.forms['theForm'].elements.length; i++)
        {
            if (document.forms['theForm'].elements[i].type=="checkbox")
            {
                if (document.forms['theForm'].elements[i].value == regionId)
                {
                    exists = true;
                    alert(region_exists);
                }
            }
        }
        // 创建checkbox
        if (!exists)
        {
            regionCell.innerHTML += "<input type='checkbox' name='regions[]' value='" + regionId + "' checked='true' /> " + regionName + "&nbsp;&nbsp;";
        }
    }

    /**
     * 配送费用计算方式
     */
    function compute_mode(shipping_code,mode)
    {
        var base_fee  = document.getElementById("base_fee");
        var step_fee  = document.getElementById("step_fee");
        var item_fee  = document.getElementById("item_fee");
        if(shipping_code == 'post_mail' || shipping_code == 'post_express')
        {
            var step_fee1  = document.getElementById("step_fee1");
        }

        if(mode == 'number')
        {
            item_fee.style.display = '';
            base_fee.style.display = 'none';
            step_fee.style.display = 'none';
            if(shipping_code == 'post_mail' || shipping_code == 'post_express')
            {
                step_fee1.style.display = 'none';
            }
        }
        else
        {
            item_fee.style.display = 'none';
            base_fee.style.display = '';
            step_fee.style.display = '';
            if(shipping_code == 'post_mail' || shipping_code == 'post_express')
            {
                step_fee1.style.display = '';
            }
        }
    }
    //-->
    {/literal}
</script>
{include file="pagefooter.htm"}
