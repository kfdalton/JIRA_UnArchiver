USE [RTMaster]
GO
/****** Object:  StoredProcedure [dbo].[JIRA_UnArchiver]   ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:		Dalton, Kevin
-- Description:	SP to unarchive issues
-- =============================================
ALTER PROCEDURE [dbo].[JIRA_UnArchiver] 
@jirainstance VARCHAR(255) = NULL
, @List VARCHAR(255) = NULL

AS
BEGIN
--DECLARE @List NVARCHAR(255)
DECLARE @ListItem NVARCHAR(255) 
DECLARE @Pos int

-- assign a list of PRoject Keys  in format 'ABC,123,kjh' to our comma delimited list variable
SET @List = 'CCTA-3183'


-- Loop while the list string still holds one or more characters
WHILE LEN(@List) > 0
Begin
       -- Get the position of the first comma (returns 0 if no commas left in string)
       SET @Pos = CHARINDEX(',', @List)

       -- Extract the list item string
       IF @Pos = 0
       Begin
               SET @ListItem = @List
       End
       ELSE
       Begin
               SET @ListItem = SUBSTRING(@List, 1, @Pos - 1)
       End
    
       PRINT '-------------Restoring -- ' + @ListItem + '-------------------'
	   PRINT ''
------------------------ONLY MODIFY THESE
	
	DECLARE @baseURL NVARCHAR(255);
	DECLARE @issueKey NVARCHAR(255);
	SET @baseURL = ('https://'+@jirainstance+'.yoururl.com'); -- TARGET INSTANCE TO UPDATE
	SET @issueKey = @ListItem
	
-------------------------
	
	DECLARE @authHeader NVARCHAR(64);
	DECLARE @contentType NVARCHAR(64);
	DECLARE @postData NVARCHAR(2000);
	DECLARE @postDataCheck NVARCHAR(2000);
	DECLARE @postDataLead NVARCHAR(2000);
	DECLARE @issueKeyName NVARCHAR(2000);
	DECLARE @urlDeleteRole NVARCHAR(2000);
	DECLARE @responseText NVARCHAR(2000);
	DECLARE @responseXML NVARCHAR(2000);
	DECLARE @ret INT;
	DECLARE @status NVARCHAR(32);
	DECLARE @statusText NVARCHAR(32);
	DECLARE @token INT;
	DECLARE @url NVARCHAR(256);
	DECLARE @data_table VARCHAR(50);
	DECLARE @PUT NVARCHAR(20);
	DECLARE @POST NVARCHAR(20);
	DECLARE @DELETE NVARCHAR(20);
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;

	SET @authHeader = N'Basic TOKEN';
	SET @contentType = N'application/json';
	SET @PUT = 'PUT';
	SET @POST = 'POST';
	SET @DELETE = 'DELETE';
	

	SET @url = @baseURL + '/rest/api/2/issue/'+ @issueKey +'/restore'
	

			EXEC @ret = sp_OACreate N'MSXML2.ServerXMLHTTP.3.0'
							   ,@token OUT;
            --print 'sp_OACreate MSXML2.ServerXMLHTTP.3.0: '
			IF @ret <> 0
				RAISERROR ('Unable to open HTTP connection.', 10, 1);

			-- Send the request.
			EXEC @ret = sp_OAMethod @token
							   ,'open'
							   ,NULL
							   ,@PUT
							   ,@url
							   ,'false';
			--print 'sp_OAMethod open: '
			EXEC @ret = sp_OAMethod @token
							   ,'setRequestHeader'
							   ,NULL
							   ,'Authorization'
							   ,@authHeader;
			--print 'sp_OAMethod setRequestHeader Authorization: ' 
			EXEC @ret = sp_OAMethod @token
							   ,'setRequestHeader'
							   ,NULL
							   ,'Content-Type'
							   ,@contentType;
			--print 'sp_OAMethod setRequestHeader content-type: ' 
			EXEC @ret = sp_OAMethod @token
							   ,'send'
							   ,NULL
							   ,@postData;

			-- Handle the response.
			--print 'sp_OAMethod send: ' 
			EXEC @ret = sp_OAGetProperty @token
									,'status'
									,@status OUT;
			--print 'sp_OAMethod status: ' 
			EXEC @ret = sp_OAGetProperty @token
									,'statusText'
									,@statusText OUT;
			--print 'sp_OAMethod statustext: ' 
			EXEC @ret = sp_OAGetProperty @token
									,'responseText'
									,@responseText OUT;
			--print 'sp_OAMethod responsetext: ' 

			-- Show the response.
			IF @status in (403)
			BEGIN
			PRINT @issuekey + ' is active issue, we can not restore it.'
			PRINT ''
			END
			IF @status = 204
			BEGIN
			PRINT @issuekey + ' has been restored.'
			PRINT ''
			END
			IF @status not in (403,204)
			BEGIN
			PRINT @postData;
			PRINT @url
			PRINT N'Status: ' + @status + ' (' + @statusText + ')';
			PRINT N'Response text: ' + @responseText;
			END

			
			
			-- Close the connection.
			EXEC @ret = sp_OADestroy @token;
			IF @ret <> 0
				RAISERROR ('Unable to close HTTP connection.', 10, 1);
			

       -- remove the list item (and trailing comma if present) from the list string
       IF @Pos = 0
       Begin
               SET @List = ''
       End
       ELSE
       Begin
               -- start substring at the character after the first comma
                SET @List = SUBSTRING(@List, @Pos + 1, LEN(@List) - @Pos)
       End
End
END
