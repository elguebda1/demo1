import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.Issue
import static groovyx.net.http.Method.*
import static groovyx.net.http.ContentType.*
import groovy.json.JsonBuilder
import groovyx.net.http.HTTPBuilder
import org.apache.log4j.Category

Category log = Category.getInstance("com.onresolve.jira.groovy")
log.setLevel(org.apache.log4j.Level.DEBUG)

Issue issue = issue

log.debug "Début de Traitement"

//***********************************************
//REST API call pour récupérer la liste des SLA liés à un ticket
//***********************************************
// Si l'adresse vient à changer il faut la changer ici
def http = new HTTPBuilder('http://10.12.1.4:8080/')
def response = http.request(GET, TEXT) {
        uri.path = 'rest/servicedeskapi/request/'+issue.key+'/sla'
        // utilisez un admin pour la connexion ou un agent Solution
        headers.'Authorization' = "Basic ${'eelguebda:Switch2021*$'.bytes.encodeBase64().toString()}"
        requestContentType = JSON
        headers.Accept = 'application/json'
    }
log.debug "Retour du call API"

//***********************************************
//Parser le résultat pour récupérer l'ID du SLA "VC"
//***********************************************
def result = new groovy.json.JsonSlurper().parse(response)
def slaName = result.values.name
def slaID
def i = 0
while (i <= slaName.size())
{
    if (slaName[i] == "VC")
    {
        slaID =  result.values.id[i]
        i++
    }   

    else
        i++
}
//***********************************************
//Deuxième call API pour récupérer le SLA Goal avec le bon ID
//***********************************************
// Si l'adresse vient à changer il faut la changer ici
http = new HTTPBuilder('http://10.12.1.4:8080/')
response = http.request(GET, TEXT) {
        uri.path = 'rest/servicedeskapi/request/'+issue.key+'/sla/'+slaID
        log.debug(uri.path)
        // utilisez un admin pour la connexion ou un agent Solution
        headers.'Authorization' = "Basic ${'eelguebda:Switch2021*$'.bytes.encodeBase64().toString()}"
        requestContentType = JSON
        headers.Accept = 'application/json'
    }
log.debug "Retour du call API"


//***********************************************
//Parser le résultat pour récupérer l'objectif du SLA
//***********************************************
 

try 
    {
        result = new groovy.json.JsonSlurper().parse(response)
        def ongoingCycle = result.ongoingCycles 
        def completedCycles = result.completedCycles 

 
        if(ongoingCycle == null )
        {
            def slaGgoalDuration = result.completedCycles.goalDuration.friendly
            def slaElapsedTime = result.completedCycles.elapsedTime.friendly
            def slaRemainingTime = result.completedCycles.remainingTime.friendly
            return  "SLA Goal Duration : " + slaGgoalDuration + " <br>"  + "temps reel de traitement : " + slaElapsedTime + "<br>" + " temps de depasement : " + slaRemainingTime + "<br>"  

        }
       
        else
        {
            def slaGgoalDuration = result.ongoingCycle.goalDuration.friendly
            def slaElapsedTime = result.ongoingCycle.elapsedTime.friendly
            def slaRemainingTime = result.ongoingCycle.remainingTime.friendly
            return  "SLA Goal Duration : " + slaGgoalDuration + " <br>"  + "temps reel de traitement : " + slaElapsedTime + "<br>" + " temps de depasement : " + slaRemainingTime + "<br>"  
        }
    }
catch(Exception e) 
    {
        log.debug("Exception:" + e)
    }

    


