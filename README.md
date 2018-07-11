# Salesforce-Project

/* -- Create a field on Account named "Number of Contacts". Populate this field with the number of contacts related to an account. --*/

trigger ContactTrigger on Contact (after insert,after update, after delete, after undelete) {

List<Contact> contacts = Trigger.isDelete ? Trigger.old : Trigger.new;
Set<Id> acctIds = new Set<Id>();
if(Trigger.isAfter && Trigger.isUpdate){
     for (Contact c : Trigger.old) {     
        if (c.AccountId != null && Trigger.newMap.get(c.Id).AccountId!= c.AccountId) {                
            acctIds.add(c.AccountId);
        }
     }
}

for (Contact c : contacts) {     
    if (c.AccountId != null) {
        acctIds.add(c.AccountId);
    }
}
List<Account> acctsToRollup = new List<Account>();
for (AggregateResult ar : [SELECT AccountId AcctId, Count(id) ContactCount FROM Contact WHERE AccountId in: acctIds GROUP BY AccountId]){
    Account a = new Account();
    a.Id = (Id) ar.get('AcctId'); 
    a.Number_of_Contacts__c   = (Integer) ar.get('ContactCount');
    acctsToRollup.add(a);
}
update acctsToRollup;
}

/--Build a basic lightning component that can query a list of 10 most recently created accounts and display it using a lightning app. ---/

//component

<aura:component controller="Account_Coutroller" implements="flexipage:availableForAllPageTypes">

<aura:attribute name="accounts" type="List" />
<aura:handler name="init" value="{!this}" action="{!c.doInit}" /> 
<table>    
    <tr>
        <td>
            <b>Name</b>
        </td>
        <td>
            <b>Created Date</b>
        </td>
       
    </tr>
    <aura:iteration items="{!v.accounts}" var="acc">
      <tr>
        <td> {!acc.Name}  </td>
        <td> {!acc.CreatedDate}  </td>
     </tr>

    </aura:iteration>
</table>
</aura:component>

//controller

({ doInit : function(component, event, helper) { helper.helperMethod(component, event, helper);

}
})

//helper

({ helperMethod : function(component, event, helper) {

    var action = component.get('c.getAccount');
    var self = this;
    action.setCallback(this, function(actionResult) {
       
     component.set('v.accounts', actionResult.getReturnValue());
    });
    $A.enqueueAction(action);
    
   	
}
})

//lightning app <aura:application > <c:Recent_Account_Comp /> </aura:application>

/------Make a basic http callout and print the result using system.debug-----/

public class BaiscCallouts { public static HttpResponse makeGetCallout() { Http http = new Http(); HttpRequest request = new HttpRequest(); request.setEndpoint('https://postman-echo.com/get?foo1=bar1&foo2=bar2'); request.setMethod('GET'); HttpResponse response = http.send(request);

    // If the request is successful, parse the JSON response.        
    if (response.getStatusCode() == 200) {
        // Deserializes the JSON string into collections of primitive data types.
        system.debug('***response.getBody()*****'+response.getBody());
        Map<String, Object> results = (Map<String, Object>) JSON.deserializeUntyped(response.getBody());
       
    }
    return response;
}
}

//response debug

{"args":{"foo1":"bar1","foo2":"bar2"},"headers":{"host":"postman-echo.com", "accept":"text/html, image/gif, image/jpeg, *; q=.2, /; 
q=.2","cache-control":"no-cache", "pragma":"no-cache","user-agent":"SFDC-Callout/43.0","x-forwarded-port":"443",
"x-forwarded-proto":"https"}, "url":"https://postman-echo.com/get?foo1=bar1&foo2=bar2"}
