<?xml version="1.0" encoding="UTF-8"?>

<mule xmlns:mulexml="http://www.mulesoft.org/schema/mule/xml"
	xmlns:http="http://www.mulesoft.org/schema/mule/http"
	xmlns:dw="http://www.mulesoft.org/schema/mule/ee/dw"
	xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation"
	xmlns:spring="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.mulesoft.org/schema/mule/xml http://www.mulesoft.org/schema/mule/xml/current/mule-xml.xsd
http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd
http://www.mulesoft.org/schema/mule/ee/dw http://www.mulesoft.org/schema/mule/ee/dw/current/dw.xsd
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-current.xsd
http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd">



<flow name="IAM_SAML_Flow22">
        <http:listener config-ref="" path="/" doc:name="HTTP"/>
        <mulexml:dom-to-xml-transformer doc:name="DOM to XML"/>
        <message-properties-transformer doc:name="Message Properties">
            <add-message-property key="Authorization" value="${Authorization}"/>
            <add-message-property key="username" value=""/>
            <add-message-property key="password" value=""/>
        </message-properties-transformer>
        <set-variable variableName="originalPayload" value="#[payload]" doc:name="Variable"/>
        <http:request config-ref="HTTPS_Request_Configuration_GW" path="/b2bi-api-gateway/saml" method="POST" followRedirects="false" doc:name="IAM SAML Service">
            <http:success-status-code-validator values="0..599"/>
        </http:request>
        <mulexml:dom-to-xml-transformer mimeType="application/xml" doc:name="DOM to XML"/>
        <set-variable variableName="token" value="#[payload]" mimeType="application/xml" doc:name="Variable"/>
        <set-payload value="#[flowVars.originalPayload]" doc:name="Set Payload"/>
        <dw:transform-message doc:name="Transform Message">
            <dw:set-payload><![CDATA[%dw 1.0
%output application/xml
%namespace ns0 http://www.w3.org/2003/05/soap-envelope
%namespace ns5 http://atlas.countrywide.com
%namespace ns4 http://www.w3.org/2005/08/addressing
%namespace ns1 http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd
%namespace ns2 urn:oasis:names:tc:SAML:2.0:assertion
%namespace ns3 http://www.w3.org/2000/09/xmldsig#
---
{
	ns0#Envelope: {
		ns0#Header: {
			ns1#Security: flowVars.token
				,
			ns4#ReplyTo: {
				ns4#Address: payload.ns0#Envelope.ns0#Header.ns4#ReplyTo.ns4#Address
			},
			ns4#To: payload.ns0#Envelope.ns0#Header.ns4#To,
			ns4#Action: payload.ns0#Envelope.ns0#Header.ns4#Action,
			ns5#RequestHeader: {
				ns5#SourceSystem: payload.ns0#Envelope.ns0#Header.ns5#RequestHeader.ns5#SourceSystem,
				ns5#MessageGenerator: payload.ns0#Envelope.ns0#Header.ns5#RequestHeader.ns5#MessageGenerator,
				ns5#RequestInfo: payload.ns0#Envelope.ns0#Header.ns5#RequestHeader.ns5#RequestInfo,
				ns5#LoginName: payload.ns0#Envelope.ns0#Header.ns5#RequestHeader.ns5#LoginName,
				ns5#EnvironmentCd: payload.ns0#Envelope.ns0#Header.ns5#RequestHeader.ns5#EnvironmentCd
			}
		},
		ns0#Body: payload.ns0#Envelope.ns0#Body
	}
}]]></dw:set-payload>
        </dw:transform-message>
        <logger message="#[payload]" level="INFO" doc:name="Logger"/>
    </flow>
    
    
</mule>
